---
name: fvtt-applications
description: This skill should be used when creating custom windows with ApplicationV2, building forms with FormApplication, using Dialog for prompts, understanding the render lifecycle, or migrating from Application v1 to ApplicationV2.
---

# Foundry VTT Applications

**Domain:** Foundry VTT Module/System Development
**Status:** Production-Ready
**Last Updated:** 2026-01-05

## Overview

Applications are the window/dialog system in Foundry. ApplicationV2 (V12+) is the modern framework replacing the legacy Application v1 (deprecated, removed in V16).

### When to Use This Skill

- Creating custom UI windows
- Building data entry forms
- Showing confirmation dialogs
- Understanding render lifecycle
- Migrating v1 applications to v2

## ApplicationV2 Basics

### Minimal Application

```javascript
const { ApplicationV2, HandlebarsApplicationMixin } = foundry.applications.api;

class MyWindow extends HandlebarsApplicationMixin(ApplicationV2) {
  static DEFAULT_OPTIONS = {
    id: "my-window",
    classes: ["my-module"],
    position: { width: 400, height: "auto" },
    window: {
      title: "My Window",
      icon: "fas fa-gear"
    }
  };

  static PARTS = {
    main: {
      template: "modules/my-module/templates/window.hbs"
    }
  };

  async _prepareContext(options) {
    return {
      message: "Hello World"
    };
  }
}

// Usage
new MyWindow().render(true);
```

### Form Application

```javascript
class MyForm extends HandlebarsApplicationMixin(ApplicationV2) {
  static DEFAULT_OPTIONS = {
    id: "my-form",
    tag: "form",  // CRITICAL for form handling
    window: { title: "Settings" },
    position: { width: 500 },
    form: {
      handler: MyForm.#onSubmit,
      submitOnChange: false,
      closeOnSubmit: true
    }
  };

  static PARTS = {
    form: {
      template: "modules/my-module/templates/form.hbs"
    }
  };

  async _prepareContext() {
    return {
      settings: this.settings
    };
  }

  static async #onSubmit(event, form, formData) {
    const data = foundry.utils.expandObject(formData.object);
    console.log("Submitted:", data);
    // Process form data
  }
}
```

## DialogV2

### Confirmation Dialog

```javascript
const confirmed = await foundry.applications.api.DialogV2.confirm({
  window: { title: "Confirm" },
  content: "<p>Delete this item?</p>"
});

if (confirmed) {
  await item.delete();
}
```

### Input Prompt

```javascript
const name = await foundry.applications.api.DialogV2.prompt({
  window: { title: "Enter Name" },
  content: "<p>What is your character's name?</p>",
  label: "Submit"
});
```

### Custom Buttons

```javascript
const result = await foundry.applications.api.DialogV2.wait({
  window: { title: "Choose Action" },
  content: "<p>What would you like to do?</p>",
  buttons: [{
    icon: "fas fa-check",
    label: "Accept",
    action: "accept"
  }, {
    icon: "fas fa-times",
    label: "Decline",
    action: "decline"
  }]
});

if (result === "accept") {
  // Handle accept
}
```

### Form Dialog

```javascript
const data = await foundry.applications.api.DialogV2.prompt({
  window: { title: "Configure" },
  content: `
    <div class="form-group">
      <label>Name</label>
      <input type="text" name="name" value="">
    </div>
    <div class="form-group">
      <label>Level</label>
      <input type="number" name="level" value="1">
    </div>
  `,
  ok: {
    callback: (event, button, dialog) => {
      const form = dialog.querySelector("form");
      return new FormDataExtended(form).object;
    }
  }
});
```

## Render Lifecycle

### Key Methods

```javascript
class MyApp extends HandlebarsApplicationMixin(ApplicationV2) {
  // Prepare data for template
  async _prepareContext(options) {
    return { items: this.items };
  }

  // Prepare data for specific part
  async _preparePartContext(partId, context) {
    if (partId === "list") {
      context.sortedItems = this.sortItems(context.items);
    }
    return context;
  }

  // After first render only
  async _onFirstRender(context, options) {
    this.setupInitialState();
  }

  // After every render
  async _onRender(context, options) {
    this.attachEventListeners();
  }
}
```

### Lifecycle Order

```
render(true) called
  → _preRender()
  → _prepareContext()
  → _preparePartContext() (per part)
  → Template rendering
  → _onFirstRender() (first time only)
  → _onRender()
  → Hook: renderMyApp
```

## Event Handling

### Static Actions (Recommended)

```javascript
class MyApp extends HandlebarsApplicationMixin(ApplicationV2) {
  static DEFAULT_OPTIONS = {
    actions: {
      delete: MyApp.#onDelete,
      edit: MyApp.#onEdit
    }
  };

  static async #onDelete(event, target) {
    const itemId = target.dataset.itemId;
    await this.deleteItem(itemId);
  }

  static #onEdit(event, target) {
    const itemId = target.dataset.itemId;
    this.editItem(itemId);
  }
}
```

Template:
```handlebars
<button type="button" data-action="delete" data-item-id="{{item.id}}">
  Delete
</button>
```

### Manual Listeners in _onRender

```javascript
async _onRender(context, options) {
  this.element.querySelector(".custom-button")
    ?.addEventListener("click", this._onCustomClick.bind(this));
}

_onCustomClick(event) {
  event.preventDefault();
  // Handle click
}
```

## Multi-Part Templates

### PARTS Configuration

```javascript
static PARTS = {
  header: {
    template: "modules/my-mod/templates/header.hbs"
  },
  tabs: {
    template: "templates/generic/tab-navigation.hbs"
  },
  content: {
    template: "modules/my-mod/templates/content.hbs",
    scrollable: [""]  // Enable scroll preservation
  },
  footer: {
    template: "modules/my-mod/templates/footer.hbs"
  }
};
```

### Tab Navigation

```javascript
static PARTS = {
  tabs: { template: "templates/generic/tab-navigation.hbs" },
  details: { template: "templates/details.hbs" },
  inventory: { template: "templates/inventory.hbs" }
};

static TABS = {
  primary: {
    tabs: [
      { id: "details", label: "Details" },
      { id: "inventory", label: "Inventory" }
    ],
    initial: "details"
  }
};

async _prepareContext(options) {
  const context = await super._prepareContext(options);
  context.tabs = this._prepareTabs();
  return context;
}

async _preparePartContext(partId, context) {
  if (["details", "inventory"].includes(partId)) {
    context.tab = context.tabs[partId];
  }
  return context;
}
```

## Legacy FormApplication (V1)

For maintenance of existing code only:

```javascript
class LegacyForm extends FormApplication {
  static get defaultOptions() {
    return foundry.utils.mergeObject(super.defaultOptions, {
      id: "legacy-form",
      title: "Legacy Form",
      template: "modules/my-mod/templates/form.hbs",
      width: 400,
      closeOnSubmit: true
    });
  }

  getData() {
    return { data: this.object };
  }

  async _updateObject(event, formData) {
    await this.object.update(formData);
  }

  activateListeners(html) {
    super.activateListeners(html);
    html.find(".button").click(this._onClick.bind(this));
  }
}
```

## Common Pitfalls

### 1. Missing tag: "form"

```javascript
// WRONG - form submission won't work
static DEFAULT_OPTIONS = {
  form: { handler: MyApp.#onSubmit }
};

// CORRECT
static DEFAULT_OPTIONS = {
  tag: "form",
  form: { handler: MyApp.#onSubmit }
};
```

### 2. Button Submits Form

```html
<!-- WRONG - triggers form submission -->
<button>Click Me</button>

<!-- CORRECT - won't submit form -->
<button type="button">Click Me</button>
```

### 3. DialogV2 Re-render

```javascript
// WRONG - DialogV2 cannot re-render
const dialog = new DialogV2({...});
await dialog.render(true);
await dialog.render(true);  // Error!

// CORRECT - use ApplicationV2 for re-renderable windows
class MyDialog extends HandlebarsApplicationMixin(ApplicationV2) {}
```

### 4. Async in _prepareContext

```javascript
// Always await async operations
async _prepareContext(options) {
  const data = await this.loadData();  // OK
  return { data };
}
```

### 5. Losing this Context

```javascript
// WRONG - loses context
this.element.addEventListener("click", this._onClick);

// CORRECT - bind context
this.element.addEventListener("click", this._onClick.bind(this));

// Or use arrow function
this.element.addEventListener("click", (e) => this._onClick(e));
```

### 6. Form Spacing Issues

```handlebars
{{!-- Use standard-form class --}}
<div class="standard-form">
  <div class="form-group">
    <label>Field</label>
    <input type="text" name="field">
  </div>
</div>
```

## Implementation Checklist

### ApplicationV2
- [ ] Extend `HandlebarsApplicationMixin(ApplicationV2)`
- [ ] Define `static DEFAULT_OPTIONS` with id, classes, position
- [ ] Define `static PARTS` for templates
- [ ] Implement `_prepareContext()` for template data
- [ ] Set `tag: "form"` for form applications
- [ ] Define `form.handler` for form submission
- [ ] Use `data-action` attributes with static actions
- [ ] Use `type="button"` on non-submit buttons

### DialogV2
- [ ] Use `DialogV2.confirm()` for yes/no prompts
- [ ] Use `DialogV2.prompt()` for text input
- [ ] Use `DialogV2.wait()` for custom buttons
- [ ] Remember DialogV2 cannot re-render

## References

- [ApplicationV2 API](https://foundryvtt.com/api/classes/foundry.applications.api.ApplicationV2.html)
- [DialogV2 API](https://foundryvtt.com/api/classes/foundry.applications.api.DialogV2.html)
- [ApplicationV2 Wiki](https://foundryvtt.wiki/en/development/api/applicationv2)
- [Conversion Guide](https://foundryvtt.wiki/en/development/guides/applicationV2-conversion-guide)
- [FormApplication (Legacy)](https://foundryvtt.com/api/classes/foundry.appv1.api.FormApplication.html)

---

**Last Updated:** 2026-01-05
**Status:** Production-Ready
**Maintainer:** ImproperSubset
