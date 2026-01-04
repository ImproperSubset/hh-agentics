# HH Agentics

HouseHudson agentic AI plugins for Claude Code.

## Installation

```bash
/plugin marketplace add ImproperSubset/hh-agentics
/plugin install cc-governance@ImproperSubset
/plugin install fvtt-dev@ImproperSubset
```

## Available Plugins

### cc-governance

Reusable governance skills and hooks for AI code assistants.

**Skills:**
| Skill | Description | Triggers |
|-------|-------------|----------|
| `cc-delegation-policy` | When/how to delegate work to subagents | "implement", "add feature", context-heavy tasks |
| `cc-development-workflow` | Agent routing, git discipline, testing | Starting features, committing, PRs |
| `cc-plugin-extensions` | Extend plugins with project-specific context | Installing plugins, project-specific paths/conventions |
| `documentation-management` | Document types, naming, lifecycle | Creating .md files, audits |
| `git-branching-strategy` | One-feature-one-branch, size limits | Starting work, branch decisions |

**Hooks:**
| Hook | Event | Purpose |
|------|-------|---------|
| Status reminder | SessionStart | Reminds to check `docs/tracking/ACTIVE.md` if it exists |
| Skill evaluation | UserPromptSubmit | Forces Claude to evaluate and load relevant skills before implementing |

### fvtt-dev

Essential skills for Foundry VTT module development.

**Skills:**
| Skill | Description | Triggers |
|-------|-------------|----------|
| `fvtt-v13-migration` | V13 patterns and migration guidance | "migrate to V13", "ESM", "DataModel", API changes |
| `fvtt-performance-safe-updates` | Multi-client update safety | Document updates, hooks, concurrent operations |
| `fvtt-version-compat` | V12/V13 API compatibility layer | Version detection, compat wrappers |
| `fvtt-data-migrations` | Schema versioning and migrations | Data upgrades, schema changes |
| `fvtt-error-handling` | Error handling patterns | Hooks.onError, notifications |

## License

MIT
