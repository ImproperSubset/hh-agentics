# Active Work - hh-agentics

## Current Focus

Adding FVTT developer skills based on Foundry VTT documentation research.

## Backlog: fvtt-dev Skills

### High-Value (recommended next)

| Skill | Topics | Status |
|-------|--------|--------|
| fvtt-hooks | Hooks.on/once/call/callAll, lifecycle, pitfalls | Done |
| fvtt-data-storage | Flags vs Settings vs Files, scope types, namespacing | Done |
| fvtt-sheets | ActorSheet/ItemSheet extension, activateListeners, render lifecycle | Done |
| fvtt-dice-rolls | Roll class, evaluation, toMessage, custom dice, getRollData | Done |
| fvtt-sockets | Multiplayer sync, socket.emit/executeAsGM patterns | Done |

### Medium-Value

| Skill | Topics | Status |
|-------|--------|--------|
| fvtt-active-effects | Effect application, timing, CUSTOM mode, duration tracking | Done |
| fvtt-canvas | PIXI.js basics, layers, placeables, render flags | Done |
| fvtt-localization | Language files, templates, JS API, pluralization | Done |
| fvtt-compendiums | Pack creation, metadata, distribution | Done |
| fvtt-combat | Combat document, initiative, combatant sorting | Done |
| fvtt-chat-messages | ChatMessage types, formatting, roll output | Done |
| fvtt-applications | FormApplication, Dialog, custom windows | Done |

## Known Issues

### Claude Code Plugin System

Filed bug reports:
- [#16288](https://github.com/anthropics/claude-code/issues/16288) - Plugin hooks not loaded from external hooks.json
- [#16289](https://github.com/anthropics/claude-code/issues/16289) - SubagentStop systemMessage not displayed in UI

Workaround: Define hooks in `~/.claude/settings.json` instead of plugins.

## Recently Completed

- 2026-01-05: Added fvtt-compendiums, fvtt-combat, fvtt-chat-messages, fvtt-applications skills (v1.8.0)
- 2026-01-05: Added fvtt-active-effects, fvtt-canvas, fvtt-localization skills (v1.7.0)
- 2026-01-04: Added fvtt-sockets skill (v1.6.0)
- 2026-01-04: Added fvtt-dice-rolls skill (v1.5.0)
- 2026-01-04: Added fvtt-sheets skill (v1.4.0)
- 2026-01-04: Added fvtt-data-storage skill (v1.3.0)
- 2026-01-04: Added fvtt-hooks skill (v1.2.0)
- 2026-01-04: Fixed cc-governance plugin config (registered hooks, removed missing skill)
- 2026-01-04: Added CLAUDE.md for repo conventions
