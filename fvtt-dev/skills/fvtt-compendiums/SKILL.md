---
name: fvtt-compendiums
description: This skill should be used when creating compendium packs, registering packs in manifests, importing/exporting documents, querying pack contents, or using the CLI for pack management and version control workflows.
---

# Foundry VTT Compendium Packs

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

Compendium packs store pre-built content (actors, items, journal entries, etc.) for distribution with modules and systems. Understanding pack management is essential for content creation.

### When to Use This Skill

- Creating content packs for distribution
- Registering packs in module/system manifests
- Importing/exporting documents programmatically
- Setting up version control workflows with CLI
- Querying and searching pack contents

## Pack Registration

### Manifest Configuration

```json
{
  "id": "my-module",
  "packs": [
    {
      "name": "monsters",
      "label": "Monsters",
      "type": "Actor",
      "path": "./packs/monsters",
      "system": "dnd5e"
    },
    {
      "name": "items",
      "label": "Magic Items",
      "type": "Item",
      "path": "./packs/items"
    }
  ]
}
```

### Valid Document Types

- `Actor` - Characters, NPCs, creatures
- `Item` - Equipment, spells, features
- `JournalEntry` - Lore, handouts
- `RollTable` - Random tables
- `Scene` - Maps and encounters
- `Macro` - Executable scripts
- `Playlist` - Audio collections
- `Cards` - Card decks
- `Adventure` - Mixed content bundles

### Directory Structure

```
my-module/
├── module.json
├── packs/
│   ├── monsters/     # LevelDB folder (V11+)
│   └── items/
└── src/
    └── packs/        # JSON/YAML source (for version control)
```

## Accessing Packs

### Get Pack Reference

```javascript
const pack = game.packs.get("my-module.monsters");

// Check properties
console.log(pack.locked);    // Edit lock status
console.log(pack.visible);   // User visibility
console.log(pack.metadata);  // Pack configuration
```

### Load Index (Lightweight)

```javascript
// Get minimal cached data
const index = await pack.getIndex();

for (const entry of index) {
  console.log(entry._id, entry.name);
}
```

### Get Single Document

```javascript
const actor = await pack.getDocument(documentId);
console.log(actor.name, actor.system);
```

### Get Multiple Documents

```javascript
// All documents (expensive for large packs)
const allDocs = await pack.getDocuments();

// Filtered query
const npcs = await pack.getDocuments({ type: "npc" });
```

### Search Contents

```javascript
const results = await pack.search({
  query: "dragon",
  fields: ["name", "system.description"]
});
```

## Import/Export

### Import Document to World

```javascript
// From pack to world
const pack = game.packs.get("my-module.monsters");
const doc = await pack.getDocument(docId);
const imported = await Actor.create(doc.toObject());
```

### Export Document to Pack

```javascript
// Requires unlocked pack
const pack = game.packs.get("my-module.monsters");

if (!pack.locked) {
  await pack.importDocument(existingActor);
}
```

### Bulk Import

```javascript
await pack.importAll({
  folderName: "Imported Monsters",
  keepId: true  // Preserve document IDs
});
```

## CLI Workflow

### Installation

```bash
npm install @foundryvtt/foundryvtt-cli --save-dev
```

### Extract for Version Control

```bash
# Unpack to JSON/YAML
fvtt package unpack -n "monsters" \
  --outputDirectory "./src/packs/monsters" \
  --yaml \
  --folders \
  --omitVolatile
```

### Repack for Distribution

```bash
# Pack back to LevelDB
fvtt package pack -n "monsters" \
  --inputDirectory "./src/packs/monsters" \
  --outputDirectory "./packs"
```

### Programmatic API

```javascript
import { extractPack, compilePack } from "@foundryvtt/foundryvtt-cli";

// Extract
await extractPack({
  packName: "monsters",
  outputDir: "./src/packs",
  yaml: true,
  omitVolatile: true
});

// Compile
await compilePack({
  packName: "monsters",
  inputDir: "./src/packs",
  outputDir: "./packs"
});
```

## Version Control Setup

### Git Attributes

```gitattributes
# Treat LevelDB as binary
packs/** binary
```

### Recommended Workflow

1. **Development**: Edit JSON/YAML in `src/packs/`
2. **Build**: Run `fvtt package pack` before commit
3. **Commit**: Include both source and compiled packs
4. **Release**: LevelDB packs ready for distribution

### Volatile Fields

These fields change on access and should be omitted:

```javascript
// Use --omitVolatile flag
_stats.createdTime
_stats.modifiedTime
_stats.lastModifiedBy
_stats.systemVersion
_stats.coreVersion
```

## Common Patterns

### Safe Pack Modification

```javascript
async function addToPack(packId, documentData) {
  const pack = game.packs.get(packId);

  if (pack.locked) {
    ui.notifications.warn("Pack is locked");
    return null;
  }

  return await pack.importDocument(
    new Actor(documentData)
  );
}
```

### Filter Index by Name

```javascript
async function findByName(packId, searchName) {
  const pack = game.packs.get(packId);
  const index = await pack.getIndex();

  return index.filter(entry =>
    entry.name.toLowerCase().includes(searchName.toLowerCase())
  );
}
```

### Import with Folder

```javascript
async function importWithFolder(packId, folderName) {
  const pack = game.packs.get(packId);

  // Create folder if needed
  let folder = game.folders.find(f =>
    f.name === folderName && f.type === pack.metadata.type
  );

  if (!folder) {
    folder = await Folder.create({
      name: folderName,
      type: pack.metadata.type
    });
  }

  // Import all to folder
  const docs = await pack.getDocuments();
  for (const doc of docs) {
    const data = doc.toObject();
    data.folder = folder.id;
    await doc.constructor.create(data);
  }
}
```

## Common Pitfalls

### 1. Forgetting Async/Await

```javascript
// WRONG - returns promise, not document
const doc = pack.getDocument(id);
console.log(doc.name);  // undefined!

// CORRECT
const doc = await pack.getDocument(id);
console.log(doc.name);  // "Dragon"
```

### 2. Modifying Locked Packs

```javascript
// WRONG - will fail silently or error
await pack.importDocument(actor);

// CORRECT - check lock first
if (!pack.locked) {
  await pack.importDocument(actor);
} else {
  ui.notifications.warn("Unlock the pack first");
}
```

### 3. Loading All Documents

```javascript
// BAD - memory issues with large packs
const all = await pack.getDocuments();

// BETTER - use index for listings
const index = await pack.getIndex();
// Only load specific documents when needed
```

### 4. User Data Overwrite

```javascript
// WARNING: Module updates overwrite pack contents
// Never store user-created content in module packs
// Use world compendiums for user content
```

### 5. Missing System Field

```javascript
// WRONG - pack won't work with system
{
  "name": "items",
  "type": "Item",
  "path": "./packs/items"
}

// CORRECT - include system for typed content
{
  "name": "items",
  "type": "Item",
  "path": "./packs/items",
  "system": "dnd5e"
}
```

### 6. Wrong Pack Path

```javascript
// V11+ uses folders, not .db files
// WRONG
"path": "./packs/monsters.db"

// CORRECT
"path": "./packs/monsters"
```

## Implementation Checklist

- [ ] Register packs in manifest with correct type
- [ ] Include `system` field for system-specific content
- [ ] Use folders (not .db files) for V11+ packs
- [ ] Set up CLI for version control workflow
- [ ] Use `--omitVolatile` when extracting for git
- [ ] Check `pack.locked` before modifications
- [ ] Use index for listings, documents for details
- [ ] Test import/export in fresh world
- [ ] Add `.gitattributes` for binary packs

## References

- [Compendium Packs Article](https://foundryvtt.com/article/compendium/)
- [CompendiumCollection API](https://foundryvtt.com/api/classes/client.CompendiumCollection.html)
- [Content Packaging Guide](https://foundryvtt.com/article/packaging-guide/)
- [V11 LevelDB Changes](https://foundryvtt.com/article/v11-leveldb-packs/)
- [CLI Package](https://www.npmjs.com/package/@foundryvtt/foundryvtt-cli)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
