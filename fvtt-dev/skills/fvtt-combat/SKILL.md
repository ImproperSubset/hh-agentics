---
name: fvtt-combat
description: This skill should be used when working with the Combat tracker, Combatant documents, initiative rolling and sorting, turn/round management, or implementing combat hooks like combatStart, combatTurn, and combatRound.
---

# Foundry VTT Combat System

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

Foundry's combat system manages turn-based encounters through Combat and Combatant documents. Understanding the combat lifecycle is essential for game system development.

### When to Use This Skill

- Implementing initiative rolling formulas
- Managing turn order and round progression
- Hooking into combat events
- Extending combat behavior for game systems
- Creating combat-related UI features

## Combat Document

### Core Properties

```javascript
game.combat            // Active combat encounter
game.combats           // All combat encounters
game.combats.active    // Currently active combat

// Combat properties
combat.round           // Current round number
combat.turn            // Current turn index
combat.active          // Is combat started?
combat.combatants      // EmbeddedCollection of Combatants
combat.combatant       // Current turn's Combatant
combat.scene           // Linked Scene
```

### Combat Methods

```javascript
// Start/end combat
await combat.startCombat();
await combat.endCombat();

// Navigation
await combat.nextTurn();
await combat.previousTurn();
await combat.nextRound();
await combat.previousRound();

// Initiative
await combat.rollInitiative(["combatantId"]);
await combat.rollAll();    // Roll for all
await combat.rollNPC();    // Roll for NPCs only
await combat.resetAll();   // Clear all initiative
```

## Combatant Document

### Core Properties

```javascript
combatant.actor        // Associated Actor
combatant.token        // Token data (not Token instance)
combatant.initiative   // Initiative value
combatant.active       // Is current turn?
combatant.defeated     // Defeated status
combatant.hidden       // Hidden from players?
combatant.players      // Owning players
```

### Combatant Methods

```javascript
// Roll initiative for this combatant
await combatant.rollInitiative();

// Get Roll without rolling
const roll = combatant.getInitiativeRoll();
const customRoll = combatant.getInitiativeRoll("1d20+5");
```

## Initiative

### Roll Initiative

```javascript
// Single combatant
await combat.rollInitiative("combatantId");

// Multiple combatants
await combat.rollInitiative(["id1", "id2"]);

// With options
await combat.rollInitiative(["id1"], {
  formula: "1d20 + @abilities.dex.mod",
  messageOptions: { flavor: "Initiative" },
  updateTurn: true
});
```

### Custom Initiative Formula

Override in your system's Combatant class:

```javascript
class MyCombatant extends Combatant {
  _getInitiativeFormula() {
    const actor = this.actor;
    if (!actor) return "1d20";

    // System-specific formula
    return `1d20 + @abilities.dex.mod + @initiative.bonus`;
  }
}

// Register
CONFIG.Combatant.documentClass = MyCombatant;
```

### Custom Sort Order

```javascript
class MyCombat extends Combat {
  _sortCombatants(a, b) {
    // Higher initiative first
    const initA = a.initiative ?? -Infinity;
    const initB = b.initiative ?? -Infinity;
    if (initA !== initB) return initB - initA;

    // Tie-breaker: alphabetical
    return a.name.localeCompare(b.name);
  }
}

CONFIG.Combat.documentClass = MyCombat;
```

## Combat Hooks

### combatStart

```javascript
Hooks.on("combatStart", (combat, updateData) => {
  console.log(`Combat started: Round ${updateData.round}`);
});
```

### combatTurn

```javascript
Hooks.on("combatTurn", (combat, updateData, updateOptions) => {
  const combatant = combat.combatants.contents[updateData.turn];
  console.log(`${combatant.name}'s turn`);

  // updateOptions.direction: 1 (forward) or -1 (backward)
});
```

### combatRound

```javascript
Hooks.on("combatRound", (combat, updateData, updateOptions) => {
  console.log(`Round ${updateData.round} started`);
});
```

### updateCombat

```javascript
Hooks.on("updateCombat", (combat, changes, options, userId) => {
  if ("turn" in changes) {
    console.log("Turn changed");
  }
  if ("round" in changes) {
    console.log("Round changed");
  }
});
```

## Extending Combat

### Workflow Methods

Override these for system-specific behavior:

```javascript
class MyCombat extends Combat {
  // Called at start of each turn
  async _onStartTurn(combatant) {
    await super._onStartTurn(combatant);

    // Decrement duration effects
    const actor = combatant.actor;
    if (actor) {
      await this._decrementEffects(actor);
    }
  }

  // Called at end of each turn
  async _onEndTurn(combatant) {
    await super._onEndTurn(combatant);

    // Process end-of-turn effects
  }

  // Called at start of each round
  async _onStartRound() {
    await super._onStartRound();

    // Reset per-round resources
    for (const c of this.combatants) {
      await c.actor?.resetRoundResources?.();
    }
  }

  // Called at end of each round
  async _onEndRound() {
    await super._onEndRound();
  }

  async _decrementEffects(actor) {
    for (const effect of actor.effects) {
      if (effect.duration.rounds) {
        const remaining = effect.duration.rounds - 1;
        if (remaining <= 0) {
          await effect.delete();
        } else {
          await effect.update({ "duration.rounds": remaining });
        }
      }
    }
  }
}

CONFIG.Combat.documentClass = MyCombat;
```

### GM-Only Execution

Workflow methods only run for one GM. Handle player-side logic with hooks:

```javascript
// This runs for all clients
Hooks.on("combatTurn", (combat, update, options) => {
  // Player-safe logic here
});
```

## Common Patterns

### Get Current Combatant

```javascript
function getCurrentCombatant() {
  const combat = game.combat;
  if (!combat?.started) return null;
  return combat.combatant;
}
```

### Check If Actor's Turn

```javascript
function isActorsTurn(actor) {
  const combat = game.combat;
  if (!combat?.started) return false;
  return combat.combatant?.actor?.id === actor.id;
}
```

### Add Token to Combat

```javascript
async function addToCombat(token) {
  let combat = game.combat;

  // Create combat if none exists
  if (!combat) {
    combat = await Combat.create({ scene: token.scene.id });
  }

  // Add combatant
  await combat.createEmbeddedDocuments("Combatant", [{
    tokenId: token.id,
    actorId: token.actor?.id,
    sceneId: token.scene.id
  }]);
}
```

### Skip Defeated Combatants

```javascript
class MyCombat extends Combat {
  async nextTurn() {
    let next = this.turn + 1;
    let round = this.round;

    // Skip defeated
    while (this.combatants.contents[next]?.defeated) {
      next++;
      if (next >= this.combatants.size) {
        next = 0;
        round++;
      }
    }

    return this.update({ turn: next, round });
  }
}
```

## Common Pitfalls

### 1. Empty Combat Errors

```javascript
// WRONG - errors with no combatants
const combat = await Combat.create({ scene: sceneId });
await combat.startCombat();  // Error!

// CORRECT - add combatants first
const combat = await Combat.create({
  scene: sceneId,
  combatants: [{ tokenId: token1.id }]
});
await combat.startCombat();
```

### 2. Token vs TokenDocument

```javascript
// combatant.token is TokenDocument data, not Token
const tokenData = combatant.token;  // Plain object

// To get actual Token instance:
const token = canvas.tokens.get(combatant.tokenId);
```

### 3. Null Initiative

```javascript
// Initiative can be null before rolling
const init = combatant.initiative;  // Could be null

// Handle null in comparisons
const sortedInit = init ?? -Infinity;
```

### 4. Workflow Method Timing

```javascript
// _onStartTurn runs AFTER the turn changes
// Use hooks if you need to act BEFORE
Hooks.on("preUpdateCombat", (combat, changes) => {
  // Runs before any combat update
});
```

### 5. Linked vs Unlinked Tokens

```javascript
// Be aware of token linking
if (combatant.token.actorLink) {
  // Changes affect the base Actor
} else {
  // Changes only affect this token's synthetic actor
}
```

### 6. Multiple GM Execution

```javascript
// Workflow methods only run for one GM
// Don't rely on them for all-client logic

// Use hooks for all-client execution
Hooks.on("combatTurn", () => {
  // Runs on all clients
});
```

## Implementation Checklist

- [ ] Register custom Combat/Combatant classes in CONFIG
- [ ] Override `_getInitiativeFormula()` for system formula
- [ ] Implement `_sortCombatants()` for custom ordering
- [ ] Use workflow methods for GM-only automation
- [ ] Use hooks for all-client logic
- [ ] Handle null initiative values
- [ ] Check `combat.started` before accessing turn data
- [ ] Consider defeated/hidden combatants
- [ ] Test with multiple GMs connected

## References

- [Combat API](https://foundryvtt.com/api/classes/foundry.documents.Combat.html)
- [Combatant API](https://foundryvtt.com/api/classes/foundry.documents.Combatant.html)
- [Combat Encounters Article](https://foundryvtt.com/article/combat/)
- [Hook Events](https://foundryvtt.com/api/modules/hookEvents.html)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
