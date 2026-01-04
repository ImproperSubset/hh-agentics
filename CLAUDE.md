# CLAUDE.md

Claude Code plugin marketplace repository.

## Repository Structure

```
├── .claude-plugin/marketplace.json  # Marketplace registry
├── cc-governance/                   # Governance skills plugin
└── fvtt-dev/                        # Foundry VTT development plugin
```

## Plugin Structure

Each plugin follows this layout:
```
plugin-name/
├── .claude-plugin/plugin.json       # Plugin manifest (name, version, skills list)
├── hooks/hooks.json                 # Optional hooks
└── skills/
    └── skill-name/
        └── SKILL.md                 # Skill content with YAML frontmatter
```

## Adding a New Skill

1. Create directory under `plugin-name/skills/`
2. Add `SKILL.md` with frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Brief description for skill selection
   ---
   ```
3. Register in plugin's `plugin.json` skills array

## Adding a New Plugin

1. Create plugin directory at repo root
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills under `skills/` subdirectory
4. Register in `.claude-plugin/marketplace.json`

## Conventions

- Skill names use kebab-case
- Skills are markdown files with practical examples and checklists
- Hooks use JSON format in `hooks/hooks.json`
- Version follows semver in plugin.json
