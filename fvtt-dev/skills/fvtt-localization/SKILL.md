---
name: fvtt-localization
description: This skill should be used when implementing internationalization (i18n), creating language files, using game.i18n.localize/format, adding template localization helpers, or following best practices for translatable strings.
---

# Foundry VTT Localization

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

Foundry VTT uses JSON language files for internationalization. All user-facing text should be localized to support translation.

### When to Use This Skill

- Creating language files for modules/systems
- Using localize/format in JavaScript code
- Adding localization to templates
- Following naming conventions for translatable strings
- Handling pluralization and interpolation

## Language File Structure

### Basic Format

```json
{
  "MYMODULE.Title": "My Module",
  "MYMODULE.Settings.Enable": "Enable Feature",
  "MYMODULE.Settings.EnableHint": "Turn this feature on or off",
  "MYMODULE.Dialog.Confirm": "Are you sure?",
  "MYMODULE.Button.Save": "Save",
  "MYMODULE.Button.Cancel": "Cancel"
}
```

### Namespace Convention

Use your package ID as prefix to avoid conflicts:

```json
{
  "MYSYSTEM.Actor.HP": "Hit Points",
  "MYSYSTEM.Actor.AC": "Armor Class",
  "MYSYSTEM.Item.Weight": "Weight"
}
```

### Document Type Labels

```json
{
  "TYPES": {
    "Actor": {
      "character": "Character",
      "npc": "Non-Player Character",
      "vehicle": "Vehicle"
    },
    "Item": {
      "weapon": "Weapon",
      "armor": "Armor",
      "spell": "Spell"
    }
  }
}
```

## Manifest Registration

### module.json / system.json

```json
{
  "id": "my-module",
  "languages": [
    {
      "lang": "en",
      "name": "English",
      "path": "lang/en.json"
    },
    {
      "lang": "es",
      "name": "Español",
      "path": "lang/es.json"
    },
    {
      "lang": "fr",
      "name": "Français",
      "path": "lang/fr.json"
    }
  ]
}
```

### Language Codes

Use ISO 639-1 (2-letter) or ISO 639-2 (3-letter) codes:
- `en` - English
- `es` - Spanish
- `fr` - French
- `de` - German
- `ja` - Japanese
- `zh` - Chinese

## JavaScript API

### game.i18n.localize()

Simple string lookup:

```javascript
const title = game.i18n.localize("MYMODULE.Title");
// Returns: "My Module"

// Missing key returns the key itself
const missing = game.i18n.localize("MYMODULE.Missing");
// Returns: "MYMODULE.Missing"
```

### game.i18n.format()

String interpolation with variables:

```json
{
  "MYMODULE.Welcome": "Welcome, {name}!",
  "MYMODULE.ItemCount": "You have {count} items",
  "MYMODULE.Comparison": "{item1} vs {item2}"
}
```

```javascript
game.i18n.format("MYMODULE.Welcome", { name: "Alice" });
// Returns: "Welcome, Alice!"

game.i18n.format("MYMODULE.ItemCount", { count: 5 });
// Returns: "You have 5 items"

game.i18n.format("MYMODULE.Comparison", {
  item1: "Sword",
  item2: "Axe"
});
// Returns: "Sword vs Axe"
```

### game.i18n.has()

Check if translation exists:

```javascript
// Check with English fallback
if (game.i18n.has("MYMODULE.Feature")) {
  // Key exists (in current language OR English)
}

// Check without fallback
if (game.i18n.has("MYMODULE.Feature", false)) {
  // Key exists in current language only
}
```

### game.i18n.getListFormatter()

Format lists according to locale:

```javascript
const formatter = game.i18n.getListFormatter({
  style: "long",       // "long", "short", "narrow"
  type: "conjunction"  // "conjunction", "disjunction"
});

formatter.format(["apples", "oranges", "bananas"]);
// English: "apples, oranges, and bananas"
// Spanish: "manzanas, naranjas y plátanos"
```

## Template Localization

### Basic Usage

```handlebars
<h1>{{localize "MYMODULE.Title"}}</h1>

<button>{{localize "MYMODULE.Button.Save"}}</button>

<!-- In attributes, use single quotes -->
<input placeholder="{{localize 'MYMODULE.Placeholder'}}">

<a title="{{localize 'MYMODULE.Tooltip'}}">Hover me</a>
```

### With Variables

```handlebars
{{localize "MYMODULE.Welcome" name=user.name}}

{{localize "MYMODULE.ItemCount" count=items.length}}
```

### Dynamic Keys

```handlebars
{{localize (concat "MYMODULE.Status." statusKey)}}
```

## Pluralization

Foundry has no built-in pluralization. Handle manually:

### Separate Keys Approach

```json
{
  "MYMODULE.Item.One": "1 item",
  "MYMODULE.Item.Many": "{count} items"
}
```

```javascript
function localizeCount(count) {
  const key = count === 1
    ? "MYMODULE.Item.One"
    : "MYMODULE.Item.Many";
  return game.i18n.format(key, { count });
}
```

### Language-Aware Pluralization

```javascript
// For complex languages, use multiple keys
const key = count === 0 ? "Zero"
          : count === 1 ? "One"
          : count < 5 ? "Few"
          : "Many";

return game.i18n.format(`MYMODULE.Items.${key}`, { count });
```

## Best Practices

### Key Naming

```json
{
  // Good - specific and hierarchical
  "MYSYS.CharacterSheet.Abilities.Strength": "Strength",
  "MYSYS.CharacterSheet.Abilities.Dexterity": "Dexterity",
  "MYSYS.Dialog.ConfirmDelete.Title": "Confirm Deletion",
  "MYSYS.Dialog.ConfirmDelete.Message": "Delete {name}?",

  // Bad - vague and conflict-prone
  "MYSYS.Label": "Label",
  "MYSYS.Title": "Title",
  "MYSYS.Button": "Button"
}
```

### Word Order for Translation

```json
{
  // Good - translator can reorder
  "MYMODULE.Message": "The {adjective} {noun} is here",

  // Bad - forces English word order
  "MYMODULE.Prefix": "The",
  "MYMODULE.Suffix": "is here"
}
```

### Context-Specific Strings

```json
{
  // Good - separate by context
  "MYSYS.Button.Save.Settings": "Save Settings",
  "MYSYS.Ability.Save.Fortitude": "Fortitude Save",

  // Bad - ambiguous
  "MYSYS.Save": "Save"
}
```

### What to Localize

**Do localize:**
- UI labels and headings
- Button text
- Dialog titles and messages
- Error/notification messages
- Placeholders and hints
- Document type names

**Don't localize:**
- User-entered content
- Code identifiers
- Technical paths/URLs
- Data that users create

## Common Patterns

### Settings Registration

```javascript
game.settings.register("my-module", "enableFeature", {
  name: game.i18n.localize("MYMODULE.Settings.Enable"),
  hint: game.i18n.localize("MYMODULE.Settings.EnableHint"),
  scope: "world",
  type: Boolean,
  default: true
});
```

### Dialog with Localization

```javascript
new Dialog({
  title: game.i18n.localize("MYMODULE.Dialog.Title"),
  content: game.i18n.format("MYMODULE.Dialog.Content", {
    name: item.name
  }),
  buttons: {
    confirm: {
      label: game.i18n.localize("MYMODULE.Button.Confirm"),
      callback: () => { /* ... */ }
    },
    cancel: {
      label: game.i18n.localize("MYMODULE.Button.Cancel")
    }
  }
}).render(true);
```

### Notification

```javascript
ui.notifications.info(
  game.i18n.format("MYMODULE.Notification.Created", {
    type: item.type,
    name: item.name
  })
);
```

## Common Pitfalls

### 1. Concatenating Strings

```javascript
// WRONG - breaks translation
const msg = game.i18n.localize("MYMOD.The") + " " +
            name + " " +
            game.i18n.localize("MYMOD.IsReady");

// CORRECT - use format with placeholders
const msg = game.i18n.format("MYMOD.ItemReady", { name });
```

### 2. Hardcoded Text

```javascript
// WRONG
ui.notifications.info("Character saved!");

// CORRECT
ui.notifications.info(game.i18n.localize("MYMOD.CharacterSaved"));
```

### 3. Missing Fallback Check

```javascript
// Be defensive with dynamic keys
const key = `MYMOD.Status.${status}`;
const label = game.i18n.has(key)
  ? game.i18n.localize(key)
  : status;  // Fallback to raw value
```

### 4. Template Quote Issues

```handlebars
<!-- WRONG - breaks attribute -->
<input title="{{localize "MYMOD.Tip"}}">

<!-- CORRECT - use single quotes inside -->
<input title="{{localize 'MYMOD.Tip'}}">
```

### 5. Forgetting Manifest Entry

```json
// Don't forget to register in manifest!
{
  "languages": [
    { "lang": "en", "name": "English", "path": "lang/en.json" }
  ]
}
```

## Directory Structure

```
my-module/
├── module.json
├── lang/
│   ├── en.json      # Required - fallback language
│   ├── es.json
│   ├── fr.json
│   └── de.json
├── templates/
│   └── sheet.hbs
└── scripts/
    └── main.js
```

## Implementation Checklist

- [ ] Create `lang/en.json` with all strings
- [ ] Register languages in manifest
- [ ] Use namespaced keys (MODNAME.Category.Key)
- [ ] Use `localize()` for simple strings
- [ ] Use `format()` for strings with variables
- [ ] Use single quotes in template attributes
- [ ] Avoid string concatenation
- [ ] Provide context-specific keys
- [ ] Handle pluralization with separate keys
- [ ] Test with different languages

## References

- [Localization API](https://foundryvtt.com/api/classes/foundry.helpers.Localization.html)
- [Localization Article](https://foundryvtt.com/article/localization/)
- [Localization Wiki](https://foundryvtt.wiki/en/development/api/localization)
- [Best Practices](https://foundryvtt.wiki/en/development/guides/localization/localization-best-practices)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
