---
name: fvtt-active-effects
description: This skill should be used when creating or applying Active Effects, working with effect changes and modes (ADD, MULTIPLY, OVERRIDE, CUSTOM), implementing duration tracking for combat, or handling effect transfer from items to actors.
---

# Foundry VTT Active Effects

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

Active Effects automatically modify Actor data through a changes array. They're the primary mechanism for buffs, debuffs, equipment bonuses, and temporary conditions.

### When to Use This Skill

- Creating equipment bonuses that modify actor stats
- Implementing buff/debuff spells with duration
- Setting up status conditions (prone, stunned, etc.)
- Using CUSTOM mode for complex calculations
- Managing effect transfer from items to actors

## ActiveEffect Structure

### Core Properties

```javascript
{
  name: "Shield of Faith",
  img: "icons/magic/defensive/shield.png",
  disabled: false,
  transfer: true,           // Transfer from item to actor
  origin: "Item.abc123",    // Source reference
  statuses: ["blessed"],    // Status identifiers
  changes: [
    {
      key: "system.attributes.ac.bonus",
      mode: 2,              // ADD
      value: "2"
    }
  ],
  duration: {
    seconds: 600,           // 10 minutes
    rounds: null,
    turns: null,
    combat: null,
    startTime: null,
    startRound: null,
    startTurn: null
  }
}
```

## Effect Modes

### CONST.ACTIVE_EFFECT_MODES

| Mode | Value | Behavior |
|------|-------|----------|
| CUSTOM | 0 | System-specific logic via hook |
| ADD | 1 | Sum numbers, concat strings |
| MULTIPLY | 2 | Multiply numeric values |
| OVERRIDE | 3 | Replace value entirely |
| UPGRADE | 4 | Use if value > current |
| DOWNGRADE | 5 | Use if value < current |

### Mode Examples

```javascript
// ADD - most common
{ key: "system.attributes.ac.bonus", mode: 2, value: "2" }

// MULTIPLY - percentage bonuses
{ key: "system.attributes.movement.walk", mode: 2, value: "1.5" }

// OVERRIDE - fixed values
{ key: "system.attributes.ac.calc", mode: 3, value: "natural" }

// UPGRADE - only if better
{ key: "system.abilities.str.value", mode: 4, value: "19" }

// DOWNGRADE - only if worse
{ key: "system.abilities.dex.value", mode: 5, value: "4" }
```

## Effect Application Timing

### prepareData Order

```javascript
// Actor.prepareData() execution:
1. this.data.reset()              // Reset to source
2. this.prepareBaseData()         // Basic setup
3. this.prepareEmbeddedDocuments() // Effects apply HERE
4. this.prepareDerivedData()      // Calculate derived values
```

**Critical:** Effects apply in step 3, so they cannot modify values created in step 4.

### Solution: Initialize in prepareBaseData

```javascript
prepareBaseData() {
  // Initialize values that effects will target
  this.system.attributes.ac.bonus = 0;
  this.system.modifiers.all = [];
}

prepareDerivedData() {
  // Use modified values here
  const ac = this.system.attributes.ac.base +
             this.system.attributes.ac.bonus;
}
```

## CUSTOM Mode

For complex calculations, use mode 0 with the `applyActiveEffect` hook:

```javascript
Hooks.on("applyActiveEffect", (actor, change, current, delta, changes) => {
  // Only handle our custom keys
  if (!change.key.startsWith("system.custom.")) return;

  switch (change.key) {
    case "system.custom.damageReduction":
      // Stack damage reduction multiplicatively
      const existing = current || 1;
      changes[change.key] = existing * (1 - delta);
      break;

    case "system.custom.attackBonus":
      // Add with diminishing returns
      const bonus = current || 0;
      changes[change.key] = bonus + Math.floor(delta / (1 + bonus / 10));
      break;
  }
});
```

### Hook Parameters

- **actor**: The Actor receiving the effect
- **change**: The EffectChangeData object
- **current**: Current value at the key
- **delta**: Parsed change value (already type-converted)
- **changes**: Accumulator - modify this to apply changes

## Duration Tracking

### Combat Duration

```javascript
const combatEffect = {
  name: "Haste",
  changes: [...],
  duration: {
    rounds: 10,
    combat: game.combat?.id,
    startRound: game.combat?.round,
    startTurn: game.combat?.turn
  }
};

await actor.createEmbeddedDocuments("ActiveEffect", [combatEffect]);
```

### World Time Duration

```javascript
const timedEffect = {
  name: "Poison",
  changes: [...],
  duration: {
    seconds: 3600,  // 1 hour
    startTime: game.time.worldTime
  }
};
```

### Checking Remaining Duration

```javascript
// Combat-based
const remaining = effect.duration.rounds -
  (game.combat.round - effect.duration.startRound);

// Time-based
const elapsed = game.time.worldTime - effect.duration.startTime;
const remaining = effect.duration.seconds - elapsed;
```

## Effect Transfer

### Legacy Mode (Default)

Effects are copied from Item to Actor when embedded:

```javascript
// Default: CONFIG.ActiveEffect.legacyTransferral = true
// Effect is duplicated to actor.effects
```

### Modern Mode (Recommended)

Effects stay on Item but apply to parent Actor:

```javascript
// In system init:
CONFIG.ActiveEffect.legacyTransferral = false;

// Benefits:
// - Edit effects without re-adding items
// - Cleaner data model
// - Effects removed when item removed
```

### Transfer Property

```javascript
{
  name: "Magic Sword Bonus",
  transfer: true,   // Apply to actor (default)
  // transfer: false  // Stay on item only
  changes: [...]
}
```

## Common Patterns

### Equipment Bonus

```javascript
const magicArmorEffect = {
  name: "+2 Armor",
  img: "icons/equipment/chest/plate.png",
  transfer: true,
  changes: [
    {
      key: "system.attributes.ac.bonus",
      mode: 2,  // ADD
      value: "2"
    }
  ]
};

await item.createEmbeddedDocuments("ActiveEffect", [magicArmorEffect]);
```

### Temporary Buff

```javascript
async function applyBless(actor) {
  await actor.createEmbeddedDocuments("ActiveEffect", [{
    name: "Bless",
    img: "icons/magic/holy/prayer.png",
    statuses: ["blessed"],
    changes: [
      { key: "system.bonuses.attack", mode: 2, value: "1d4" },
      { key: "system.bonuses.save", mode: 2, value: "1d4" }
    ],
    duration: {
      rounds: 10,
      combat: game.combat?.id,
      startRound: game.combat?.round
    }
  }]);
}
```

### Status Condition

```javascript
const proneEffect = {
  name: "Prone",
  img: "icons/svg/falling.svg",
  statuses: ["prone"],
  changes: [
    { key: "system.attributes.ac.bonus", mode: 2, value: "-4" }
  ],
  flags: {
    core: { statusId: "prone" }
  }
};
```

### Formula-Based Value

```javascript
{
  key: "system.attributes.ac.bonus",
  mode: 2,
  value: "@abilities.dex.mod"  // Roll data reference
}
```

## Common Pitfalls

### 1. Targeting Derived Values

```javascript
// WRONG - calculated in prepareDerivedData, too late
{ key: "system.attributes.ac.value", ... }

// CORRECT - target the bonus that feeds into calculation
{ key: "system.attributes.ac.bonus", ... }
```

### 2. Modifying Items Directly

```javascript
// WRONG - effects cannot modify embedded items
{ key: "items.0.system.damage", ... }

// CORRECT - modify actor data only
{ key: "system.bonuses.weaponDamage", ... }
```

### 3. Forgetting to Initialize

```javascript
// WRONG - bonus undefined, effect fails
prepareDerivedData() {
  this.system.ac.total = this.system.ac.base + this.system.ac.bonus;
}

// CORRECT - initialize in prepareBaseData
prepareBaseData() {
  this.system.ac.bonus = 0;
}
```

### 4. String vs Number Values

```javascript
// Effect values are always strings in the changes array
{ key: "system.hp.bonus", mode: 2, value: "10" }

// ADD mode parses to number automatically
// For CUSTOM mode, parse manually:
const numValue = Number(change.value);
```

### 5. Checking Effect Active State

```javascript
// Always check before processing
for (const effect of actor.effects) {
  if (!effect.active) continue;  // Skip disabled
  // Process effect...
}

// Or use the filtered collection
for (const effect of actor.appliedEffects) {
  // Only active effects
}
```

### 6. Duration Display Lag

```javascript
// UI may not update in real-time
// Close/reopen sheet to see accurate duration
// Or force refresh:
await effect.updateDuration();
```

## Finding Valid Keys

```javascript
// Discover available data paths:
const paths = Object.keys(
  foundry.utils.flattenObject(game.system.model.Actor.character)
);
console.log(paths);

// Or inspect a specific actor:
console.log(Object.keys(
  foundry.utils.flattenObject(actor.system)
));
```

## Implementation Checklist

- [ ] Initialize effect targets in `prepareBaseData()`
- [ ] Use correct mode for the modification type
- [ ] Set `transfer: true` for item effects that apply to actors
- [ ] Include duration for temporary effects
- [ ] Add statuses array for conditions
- [ ] Implement CUSTOM mode hook for complex logic
- [ ] Check `effect.active` before processing
- [ ] Test with multiple stacking effects
- [ ] Verify effects removed when source removed

## References

- [ActiveEffect API](https://foundryvtt.com/api/classes/foundry.documents.ActiveEffect.html)
- [Active Effects Primer](https://foundryvtt.wiki/en/development/guides/active-effects)
- [V11 Active Effects Changes](https://foundryvtt.com/article/v11-active-effects/)
- [applyActiveEffect Hook](https://foundryvtt.com/api/functions/hookEvents.applyActiveEffect.html)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
