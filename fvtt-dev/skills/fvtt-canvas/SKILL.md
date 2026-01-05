---
name: fvtt-canvas
description: This skill should be used when working with the Foundry canvas, PIXI.js rendering, canvas layers, placeable objects (tokens, tiles, drawings), render flags for performance, or canvas lifecycle hooks like canvasReady.
---

# Foundry VTT Canvas & PIXI.js

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

The Foundry canvas is a WebGL-powered HTML5 canvas using PIXI.js for rendering. Understanding the layer architecture and PIXI basics is essential for visual customizations.

### When to Use This Skill

- Adding custom visual elements to the canvas
- Extending token/tile rendering
- Working with canvas layers
- Optimizing render performance
- Handling canvas lifecycle events

## Canvas Architecture

### Layer Hierarchy (Bottom to Top)

**Primary Group:**
- Background Layer - Scene backdrop
- Token Layer - Characters and creatures
- Tile Layer - Props and decorations
- Foreground Layer - Overlay images
- Weather Layer - Environmental effects
- Effects/Lighting - Vision and lighting

**Interface Group:**
- Walls Layer - Movement/sight blocking
- Sounds Layer - Audio zones
- Drawings Layer - User markup
- Templates Layer - Spell areas
- Notes Layer - Journal pins
- Controls Layer - Selection UI
- Grid Layer - Grid overlay

### Accessing Layers

```javascript
canvas.tokens      // TokenLayer
canvas.tiles       // TileLayer
canvas.drawings    // DrawingsLayer
canvas.templates   // TemplatesLayer
canvas.walls       // WallsLayer
canvas.lighting    // LightingLayer
canvas.sounds      // SoundsLayer
canvas.notes       // NotesLayer
canvas.grid        // GridLayer

canvas.primary     // PrimaryCanvasGroup
canvas.interface   // InterfaceCanvasGroup
canvas.environment // EnvironmentCanvasGroup
```

## PIXI.js Basics

### Containers

Group objects together:

```javascript
const group = new PIXI.Container();
group.addChild(sprite1);
group.addChild(sprite2);

// Position the group (moves all children)
group.position.set(100, 100);
group.rotation = Math.PI / 4;

canvas.tokens.addChild(group);
```

### Sprites

Display images:

```javascript
const sprite = PIXI.Sprite.from("path/to/image.png");

// Use anchor for rotation center (0-1 percentage)
sprite.anchor.set(0.5);  // Center

sprite.position.set(100, 100);
sprite.width = 50;
sprite.height = 50;
sprite.rotation = Math.PI / 4;  // Radians
sprite.alpha = 0.8;
sprite.tint = 0xFF0000;  // Red tint
```

### Graphics

Draw shapes programmatically:

```javascript
const graphics = new PIXI.Graphics();

// Filled rectangle
graphics.beginFill(0x0000FF, 0.5);  // Blue, 50% alpha
graphics.drawRect(0, 0, 100, 100);
graphics.endFill();

// Stroked circle
graphics.lineStyle(2, 0xFF0000);  // 2px red line
graphics.drawCircle(50, 50, 25);

// Polygon
graphics.beginFill(0x00FF00);
graphics.drawPolygon([0, 0, 50, 100, 100, 0]);
graphics.endFill();

canvas.drawings.addChild(graphics);
```

### Visual Effects

```javascript
// Transparency
sprite.alpha = 0.5;

// Color tint (multiply)
sprite.tint = 0xFF0000;  // Red
sprite.tint = 0xFFFFFF;  // No change (white)

// Blend modes
sprite.blendMode = PIXI.BLEND_MODES.ADD;       // Glow effect
sprite.blendMode = PIXI.BLEND_MODES.MULTIPLY;  // Darken
sprite.blendMode = PIXI.BLEND_MODES.SCREEN;    // Lighten

// Filters (use sparingly - performance impact)
const blur = new PIXI.BlurFilter();
blur.blur = 10;
sprite.filters = [blur];
```

### Masking

```javascript
// Graphics mask
const mask = new PIXI.Graphics();
mask.beginFill(0xFFFFFF);
mask.drawCircle(50, 50, 50);
mask.endFill();

container.mask = mask;
container.addChild(contentToMask);
```

## Placeable Objects

### Common Placeables

- **Token** - Actor representation
- **Tile** - Static artwork
- **Drawing** - User shapes
- **Note** - Journal pin
- **Wall** - Blocking segment
- **AmbientLight** - Light source
- **AmbientSound** - Audio emitter
- **MeasuredTemplate** - Area indicator

### PlaceableObject Properties

```javascript
const token = canvas.tokens.get(tokenId);

token.document    // TokenDocument
token.scene       // Parent Scene
token.isOwner     // Ownership check

// Permission checks
token.can("update")
token.can("delete")
token.can("control")
```

### PlaceableObject Methods

```javascript
// Control
token.control();           // Select
token.release();           // Deselect
await token.rotate(45);    // Rotate by degrees

// Rendering
await token.draw();        // Full redraw
token.refresh();           // Incremental update
```

## Render Flags

Optimize updates by specifying what changed:

### Token Render Flags

```javascript
token.renderFlags.set({
  refreshPosition: true,    // X/Y changed
  refreshSize: true,        // Width/height changed
  refreshRotation: true,    // Angle changed
  refreshBars: true,        // HP bars changed
  refreshEffects: true,     // Status icons changed
  refreshBorder: true,      // Selection border
  refreshVisibility: true,  // Vision state
  refreshElevation: true,   // Z-axis display
  refreshNameplate: true,   // Name display
  redraw: true              // Complete redraw
});
```

### Why Use Render Flags

```javascript
// BAD - full redraw every time
token.draw();
token.draw();
token.draw();

// GOOD - batch incremental updates
token.renderFlags.set({ refreshPosition: true });
token.renderFlags.set({ refreshBars: true });
// Updates happen efficiently in next render cycle
```

## Canvas Lifecycle

### Initialization Order

```
init
  → setup
    → canvasConfig
      → canvasInit
        → ready
          → canvasReady
```

### Key Hooks

```javascript
// Canvas fully ready - safe to access all layers
Hooks.on("canvasReady", (canvas) => {
  console.log("Scene:", canvas.scene.name);
  console.log("Tokens:", canvas.tokens.placeables.length);
});

// Canvas being torn down
Hooks.on("canvasTearDown", (canvas) => {
  // Clean up custom elements
});

// Canvas panned/zoomed
Hooks.on("canvasPan", (canvas, position) => {
  console.log("New center:", position.x, position.y);
  console.log("Scale:", position.scale);
});
```

### Waiting for Canvas

```javascript
// Promise-based
await canvas.ready;

// Hook-based
Hooks.once("canvasReady", () => {
  // Safe to interact
});
```

## Common Patterns

### Add Custom Layer Element

```javascript
Hooks.on("canvasReady", () => {
  const marker = new PIXI.Graphics();
  marker.beginFill(0xFF0000);
  marker.drawCircle(0, 0, 20);
  marker.endFill();
  marker.position.set(500, 500);

  canvas.interface.addChild(marker);
});
```

### Extend Token Rendering

```javascript
class CustomToken extends Token {
  async _draw() {
    await super._draw();

    // Add custom aura
    const aura = new PIXI.Graphics();
    aura.beginFill(0x00FF00, 0.2);
    aura.drawCircle(0, 0, this.w);
    aura.endFill();

    this.addChildAt(aura, 0);  // Behind token
  }
}

// Register
CONFIG.Token.objectClass = CustomToken;
```

### Coordinate Conversion

```javascript
// Client (viewport) to canvas coordinates
const canvasCoords = canvas.canvasCoordinatesFromClient({
  x: event.clientX,
  y: event.clientY
});

// Canvas to client coordinates
const clientCoords = canvas.clientCoordinatesFromCanvas({
  x: 500,
  y: 500
});
```

### Pan and Zoom

```javascript
// Instant pan
await canvas.pan({ x: 1000, y: 1000 });

// Animated pan
await canvas.animatePan({
  x: 1000,
  y: 1000,
  scale: 1.5,
  duration: 1000
});

// Center on controlled token
await canvas.recenter();
```

## Common Pitfalls

### 1. Accessing Canvas Before Ready

```javascript
// WRONG - canvas not initialized
Hooks.on("init", () => {
  canvas.tokens.placeables;  // undefined!
});

// CORRECT - wait for canvasReady
Hooks.on("canvasReady", () => {
  canvas.tokens.placeables;  // works
});
```

### 2. Direct DOM Manipulation

```javascript
// WRONG - breaks PIXI rendering
element.style.transform = "rotate(45deg)";

// CORRECT - use PIXI properties
sprite.angle = 45;  // degrees
sprite.rotation = Math.PI / 4;  // radians
```

### 3. Wrong Canvas Group

```javascript
// WRONG - bypasses layer hierarchy
canvas.app.stage.addChild(myGraphics);

// CORRECT - add to appropriate group
canvas.interface.addChild(myGraphics);
```

### 4. Excessive Filters

```javascript
// BAD - major performance hit
object.filters = [blur, color, displacement, glow];

// BETTER - minimal filters, combine effects
object.filters = [combinedEffect];
```

### 5. Not Cleaning Up

```javascript
// Remember to remove custom elements
Hooks.on("canvasTearDown", () => {
  myCustomElement.destroy();
});
```

### 6. Forgetting Anchor/Pivot

```javascript
// Rotation around top-left (default)
sprite.rotation = Math.PI / 4;

// Rotation around center (usually desired)
sprite.anchor.set(0.5);
sprite.rotation = Math.PI / 4;
```

## Implementation Checklist

- [ ] Wait for `canvasReady` before canvas access
- [ ] Add elements to correct canvas group/layer
- [ ] Use PIXI properties, not DOM manipulation
- [ ] Set anchor/pivot for rotation center
- [ ] Use render flags for efficient updates
- [ ] Clean up custom elements on `canvasTearDown`
- [ ] Use filters sparingly
- [ ] Test at different zoom levels
- [ ] Handle scene changes gracefully

## References

- [Canvas API](https://foundryvtt.com/api/classes/foundry.canvas.Canvas.html)
- [Canvas Wiki](https://foundryvtt.wiki/en/development/api/canvas)
- [PIXI.js Guide](https://foundryvtt.wiki/en/development/guides/pixi)
- [PlaceableObject API](https://foundryvtt.com/api/classes/foundry.canvas.placeables.PlaceableObject.html)
- [Token API](https://foundryvtt.com/api/classes/foundry.canvas.placeables.Token.html)
- [PixiJS Documentation](https://pixijs.io/guides/)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
