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
| fvtt-active-effects | Effect application, timing, CUSTOM mode, duration tracking | Pending |
| fvtt-canvas | PIXI.js basics, layers, placeables, render flags | Pending |
| fvtt-localization | Language files, templates, JS API, pluralization | Pending |
| fvtt-compendiums | Pack creation, metadata, distribution | Pending |
| fvtt-combat | Combat document, initiative, combatant sorting | Pending |
| fvtt-chat-messages | ChatMessage types, formatting, roll output | Pending |
| fvtt-applications | FormApplication, Dialog, custom windows | Pending |

## Known Issues

### Claude Code Plugin System

Filed bug reports:
- [#16288](https://github.com/anthropics/claude-code/issues/16288) - Plugin hooks not loaded from external hooks.json
- [#16289](https://github.com/anthropics/claude-code/issues/16289) - SubagentStop systemMessage not displayed in UI

Workaround: Define hooks in `~/.claude/settings.json` instead of plugins.

## Recently Completed

- 2026-01-04: Added fvtt-sockets skill (v1.6.0)
- 2026-01-04: Added fvtt-dice-rolls skill (v1.5.0)
- 2026-01-04: Added fvtt-sheets skill (v1.4.0)
- 2026-01-04: Added fvtt-data-storage skill (v1.3.0)
- 2026-01-04: Added fvtt-hooks skill (v1.2.0)
- 2026-01-04: Fixed cc-governance plugin config (registered hooks, removed missing skill)
- 2026-01-04: Added CLAUDE.md for repo conventions
