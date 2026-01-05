---
name: fvtt-chat-messages
description: This skill should be used when creating chat messages, sending roll results to chat, configuring speakers, implementing whispers and roll modes, or hooking into chat message rendering.
---

# Foundry VTT Chat Messages

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

ChatMessage documents display in the chat log and handle rolls, whispers, and player communication. Understanding message creation and hooks is essential for game system features.

### When to Use This Skill

- Creating chat messages programmatically
- Sending dice roll results to chat
- Configuring message speakers
- Implementing whispers and roll modes
- Adding custom rendering to messages

## ChatMessage Structure

### Core Fields

```javascript
{
  _id: "documentId",
  author: "userId",           // Who created the message
  content: "<p>HTML</p>",     // Message content
  flavor: "Roll description", // Flavor text for rolls
  speaker: {                  // Who is "speaking"
    scene: "sceneId",
    actor: "actorId",
    token: "tokenId",
    alias: "Display Name"
  },
  whisper: ["userId"],        // Private recipients
  blind: false,               // GM-only visibility
  rolls: [],                  // Dice roll data
  sound: "audio/path.ogg",    // Sound to play
  flags: {}                   // Custom data
}
```

## Creating Messages

### Simple Text Message

```javascript
await ChatMessage.create({
  content: "Hello, world!"
});
```

### Message with Speaker

```javascript
await ChatMessage.create({
  content: "I attack the dragon!",
  speaker: ChatMessage.getSpeaker({ actor: myActor })
});
```

### HTML Content

```javascript
await ChatMessage.create({
  content: `
    <h2>Critical Hit!</h2>
    <p>You deal <strong>24</strong> damage.</p>
  `,
  speaker: ChatMessage.getSpeaker({ token: myToken })
});
```

### With Custom Flags

```javascript
await ChatMessage.create({
  content: "Attack roll",
  flags: {
    "my-module": {
      rollType: "attack",
      targetId: target.id
    }
  }
});
```

## Roll Messages

### Using Roll.toMessage() (Recommended)

```javascript
const roll = new Roll("1d20 + @mod", { mod: 5 });
await roll.evaluate();

await roll.toMessage({
  speaker: ChatMessage.getSpeaker({ actor }),
  flavor: "Attack Roll"
});
```

### With Roll Mode

```javascript
await roll.toMessage({
  speaker: ChatMessage.getSpeaker({ actor }),
  flavor: "Stealth Check"
}, {
  rollMode: game.settings.get("core", "rollMode")
});
```

### Multiple Rolls

```javascript
const attackRoll = new Roll("1d20 + 5");
const damageRoll = new Roll("2d6 + 3");

await attackRoll.evaluate();
await damageRoll.evaluate();

await ChatMessage.create({
  speaker: ChatMessage.getSpeaker({ actor }),
  flavor: "Attack and Damage",
  rolls: [attackRoll, damageRoll]
});
```

## Speaker Configuration

### getSpeaker()

```javascript
// From controlled token (default)
const speaker = ChatMessage.getSpeaker();

// From specific actor
const speaker = ChatMessage.getSpeaker({ actor: myActor });

// From specific token
const speaker = ChatMessage.getSpeaker({ token: myToken });

// Custom alias
const speaker = ChatMessage.getSpeaker({ alias: "The Narrator" });
```

### Speaker Structure

```javascript
{
  scene: "sceneId",      // Scene where speaker is
  actor: "actorId",      // Actor document ID
  token: "tokenId",      // Token document ID
  alias: "Display Name"  // Fallback name
}
```

### Get Speaker's Actor

```javascript
const actor = ChatMessage.getSpeakerActor(message.speaker);
```

## Whispers and Roll Modes

### Roll Modes

| Mode | Visibility | Command |
|------|------------|---------|
| Public | Everyone | `/publicroll` |
| GM | Roller + GMs | `/gmroll` |
| Blind | GMs only | `/blindroll` |
| Self | Roller only | `/selfroll` |

### Apply Roll Mode

```javascript
// Get current setting
const rollMode = game.settings.get("core", "rollMode");

// Apply to roll
await roll.toMessage({
  speaker: ChatMessage.getSpeaker({ actor })
}, {
  rollMode: rollMode
});
```

### Whisper to Specific Users

```javascript
// Single user
await ChatMessage.create({
  content: "Secret message",
  whisper: [targetUserId]
});

// Multiple users
await ChatMessage.create({
  content: "Group secret",
  whisper: [user1Id, user2Id]
});

// All GMs
await ChatMessage.create({
  content: "GM only",
  whisper: game.users.filter(u => u.isGM).map(u => u.id)
});
```

### Blind Messages

```javascript
// GM sees content, others see "???"
await ChatMessage.create({
  content: "Secret roll result: 15",
  blind: true,
  whisper: game.users.filter(u => u.isGM).map(u => u.id)
});
```

## Chat Hooks

### renderChatMessageHTML (V13+)

```javascript
Hooks.on("renderChatMessageHTML", (message, html, context) => {
  // message: ChatMessage document
  // html: HTMLElement
  // context: Rendering context

  // Add custom styling
  if (message.flags["my-module"]?.critical) {
    html.classList.add("critical-hit");
  }

  // Add buttons
  const button = document.createElement("button");
  button.textContent = "Apply Damage";
  button.addEventListener("click", () => applyDamage(message));
  html.querySelector(".message-content").append(button);
});
```

### preCreateChatMessage

```javascript
Hooks.on("preCreateChatMessage", (message, data, options, userId) => {
  // Modify before creation
  message.updateSource({
    content: data.content + " (modified)"
  });

  // Return false to cancel
  return true;
});
```

### createChatMessage

```javascript
Hooks.on("createChatMessage", (message, options, userId) => {
  // After creation, for all clients
  console.log("New message:", message.content);
});
```

### chatMessage

```javascript
Hooks.on("chatMessage", (chatLog, messageText, chatData) => {
  // When user sends message via input
  // Return false to prevent default handling

  if (messageText.startsWith("/custom")) {
    handleCustomCommand(messageText);
    return false;
  }
});
```

## Common Patterns

### Roll with Button

```javascript
async function attackRoll(actor, target) {
  const roll = new Roll("1d20 + @mod", actor.getRollData());
  await roll.evaluate();

  await roll.toMessage({
    speaker: ChatMessage.getSpeaker({ actor }),
    flavor: `Attack vs ${target.name}`,
    flags: {
      "my-system": {
        type: "attack",
        targetId: target.id,
        total: roll.total
      }
    }
  });
}

// Handle button clicks
Hooks.on("renderChatMessageHTML", (message, html) => {
  const flags = message.flags["my-system"];
  if (flags?.type !== "attack") return;

  html.querySelector(".apply-damage")?.addEventListener("click", () => {
    const target = game.actors.get(flags.targetId);
    // Apply damage logic
  });
});
```

### Collapsible Details

```javascript
await ChatMessage.create({
  content: `
    <div class="roll-result">
      <h3>Attack Roll: 18</h3>
      <details>
        <summary>Details</summary>
        <p>Base: 1d20 = 13</p>
        <p>Modifier: +5</p>
      </details>
    </div>
  `
});
```

### Sound with Message

```javascript
await ChatMessage.create({
  content: "The bell tolls...",
  sound: "sounds/bell.ogg"
});
```

## Common Pitfalls

### 1. Forgetting Roll Evaluation

```javascript
// WRONG - total is undefined
const roll = new Roll("1d20");
await roll.toMessage();  // roll.total undefined!

// CORRECT
const roll = new Roll("1d20");
await roll.evaluate();
await roll.toMessage();
```

### 2. Wrong Speaker Token

```javascript
// WRONG - uses first controlled token
ChatMessage.getSpeaker();

// CORRECT - specify the token
ChatMessage.getSpeaker({ token: specificToken });
```

### 3. Whisper vs Roll Mode Conflict

```javascript
// Roll messages override whisper with rollMode
// Use rollMode for roll messages:
await roll.toMessage({}, {
  rollMode: "gmroll"  // Not whisper: [...]
});
```

### 4. Ignoring Roll Mode Setting

```javascript
// WRONG - always public
await roll.toMessage();

// CORRECT - respect user setting
await roll.toMessage({}, {
  rollMode: game.settings.get("core", "rollMode")
});
```

### 5. Message Update Timing

```javascript
// Updating too fast causes UI issues
// Wait for notification to fade (~3 seconds)
const msg = await ChatMessage.create({ content: "Loading..." });
setTimeout(() => {
  msg.update({ content: "Done!" });
}, 3500);
```

### 6. Not Checking Visibility

```javascript
// Check if message is visible to current user
if (!message.visible) return;

// Check if content is visible (not just presence)
if (message.isContentVisible) {
  // Safe to read content
}
```

## Implementation Checklist

- [ ] Use `ChatMessage.getSpeaker()` for proper speaker
- [ ] Always `await roll.evaluate()` before toMessage
- [ ] Respect `game.settings.get("core", "rollMode")`
- [ ] Use `renderChatMessageHTML` hook for customization
- [ ] Store custom data in flags, not content
- [ ] Handle whisper recipients appropriately
- [ ] Test all roll modes (public, GM, blind, self)
- [ ] Add sound effects where appropriate

## References

- [ChatMessage API](https://foundryvtt.com/api/classes/foundry.documents.ChatMessage.html)
- [Chat Messages Article](https://foundryvtt.com/article/chat/)
- [Roll API](https://foundryvtt.com/api/classes/foundry.dice.Roll.html)
- [renderChatMessageHTML Hook](https://foundryvtt.com/api/functions/hookEvents.renderChatMessageHTML.html)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
