---
trigger: always_on
description: CSS component library for SAP Design System. Framework-agnostic, works with any web technology.
---

# fundamental-styles

CSS component library for SAP Design System. Framework-agnostic, works with any web technology.

## For AI Agents

**Component documentation is auto-generated from Storybook stories and available in structured markdown:**

- **📚 Complete Component Index**: [docs/components/README.md](docs/components/README.md) - All 140+ components
- **📖 Per-Component Docs**: `docs/components/{component-name}.md` - Detailed examples with modifiers
- **🔧 Integration Guide**: [docs/AI_USAGE.md](docs/AI_USAGE.md) - How to use these docs in AI tools
- **📋 Schemas**: `docs/schemas/*.json` - Component structure definitions
- **♿ Accessibility**: `docs/accessibility.json` - ARIA patterns for all components

**When generating HTML/CSS, always reference the component-specific markdown file for:**
- Complete list of modifiers (`fd-{component}--{modifier}`)
- State classes (`is-error`, `is-disabled`, etc.)
- Correct HTML structure and nesting
- Accessibility requirements (ARIA attributes)

**Example workflow:**
1. User asks for a button → Read [docs/components/button.md](docs/components/button.md)
2. Find the right modifier (e.g., `fd-button--emphasized` for primary action)
3. Generate HTML with proper structure and ARIA attributes

**Using the MCP Server:**
If you have the `fundamental-styles` MCP server available, use these tools:
- `get_component_markdown` - Get complete markdown documentation with all HTML examples
- `get_component_html` - Get specific HTML variants
- `list_components` - Browse all available components
- `search_components` - Find components by keyword

**Using Skills:**
Type `/component-docs` to browse component documentation interactively.

## Quick Lookup - "I need a..."

| Need | Component | Class |
|------|-----------|-------|
| Button | button | `fd-button` |
| Text input | input | `fd-input` |
| Dropdown/Select | select | `fd-select` |
| Checkbox | checkbox | `fd-checkbox` |
| Radio button | radio | `fd-radio` |
| Toggle switch | switch | `fd-switch` |
| Link | link | `fd-link` |
| Modal/Popup | dialog | `fd-dialog` |
| Tooltip | popover | `fd-popover` |
| Dropdown menu | menu | `fd-menu` |
| Data table | table | `fd-table` |
| List of items | list | `fd-list` |
| Card/Tile | card | `fd-card` |
| Tabs | tabs | `fd-tabs` |
| Alert/Banner | alert, message-strip | `fd-alert`, `fd-message-strip` |
| Loading spinner | busy-indicator | `fd-busy-indicator` |
| Form field | form-item | `fd-form-item` |
| Icon | icon | `sap-icon--{name}` |

## Packages

| Package | Purpose | Install |
|---------|---------|---------|
| `fundamental-styles` | 120 UI components | `npm i fundamental-styles` |
| `@fundamental-styles/common-css` | Utility classes (margins, flex) | `npm i @fundamental-styles/common-css` |

## Quick Start

```html
<!-- 1. Theme (required) -->
<link href="node_modules/fundamental-styles/dist/theming/sap_horizon.css" rel="stylesheet">

<!-- 2. Components (pick what you need) -->
<link href="node_modules/fundamental-styles/dist/button.css" rel="stylesheet">
<link href="node_modules/fundamental-styles/dist/input.css" rel="stylesheet">
```

## Component Classes (fd-*)

**BEM Pattern**: `fd-{component}`, `fd-{component}--{modifier}`, `fd-{component}__{element}`

### Buttons
```html
<button class="fd-button">Default</button>
<button class="fd-button fd-button--emphasized">Primary Action</button>
<button class="fd-button fd-button--positive">Approve</button>
<button class="fd-button fd-button--negative">Reject</button>
<button class="fd-button fd-button--transparent">Ghost/Tertiary</button>
<button class="fd-button fd-button--compact">Compact Size</button>

<!-- Button with icon -->
<button class="fd-button">
  <span class="sap-icon--add"></span> Add Item
</button>

<!-- Icon-only button -->
<button class="fd-button" aria-label="Settings">
  <span class="sap-icon--settings"></span>
</button>

<!-- Disabled state -->
<button class="fd-button" disabled aria-disabled="true">Disabled</button>
```

### Form Inputs
```html
<!-- Text input -->
<input class="fd-input" type="text" placeholder="Enter text">
<input class="fd-input fd-input--compact" type="text">

<!-- States -->
<input class="fd-input is-error" type="text">     <!-- Invalid -->
<input class="fd-input is-success" type="text">   <!-- Valid -->
<input class="fd-input is-warning" type="text">   <!-- Warning -->
<input class="fd-input" disabled>                 <!-- Disabled -->
<input class="fd-input" readonly>                 <!-- Read-only -->

<!-- Complete form field -->
<div class="fd-form-item">
  <label class="fd-form-label fd-form-label--required" for="name">Name</label>
  <input class="fd-input" id="name" type="text">
</div>

<!-- With error message -->
<div class="fd-form-item">
  <label class="fd-form-label" for="email">Email</label>
  <input class="fd-input is-error" id="email" type="email" aria-describedby="email-error">
  <span class="fd-form-message fd-form-message--error" id="email-error">Invalid email</span>
</div>
```

### Select/Dropdown
```html
<div class="fd-select">
  <div class="fd-select__control" role="combobox" aria-expanded="false" aria-haspopup="listbox" tabindex="0">
    <span class="fd-select__text-content">Select an option</span>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/fundamental-styles](https://github.com/SAP/fundamental-styles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
