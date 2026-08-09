---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SaladUI is a Phoenix LiveView UI component library inspired by shadcn/ui. It provides 40+ accessible, customizable components combining server-side rendering with client-side interactivity through a hybrid architecture using Phoenix Function Components, JavaScript State Machines, and LiveView Hooks.

## Development Commands

### Testing
```bash
# Run all tests
mix test

# Run tests with color output (default alias)
mix test

# Run tests with coverage
mix coveralls

# Run tests in watch mode
mix test.watch
```

### Code Quality
```bash
# Format code (uses Phoenix.LiveView.HTMLFormatter and Styler)
mix format

# Run static analysis
mix credo

# Run full audit (format + credo + coverage)
mix audit
```

### Storybook Development
```bash
# Start the storybook Phoenix server for component development
cd storybook
mix phx.server
# Access at http://localhost:4000
```

### Installation Tasks
```bash
# Quick setup (library mode) - configures Tailwind and JS
mix salad.setup

# Full installation (local customization) - copies components locally
mix salad.install
# With options:
mix salad.install --prefix MyUI --color-scheme slate
```

## Architecture

### Component Structure

SaladUI uses a three-layer architecture:

1. **Elixir Layer** (`lib/salad_ui/*.ex`):
   - Phoenix Function Components that render HEEx templates
   - Each component module uses `use SaladUI, :component` which imports helpers and Phoenix.Component
   - Components defined with `attr`, `slot`, and function components
   - Use `TwMerge` for Tailwind class merging via `classes/1` helper
   - Support for variants through `button_variant/1` and `variant_class/2` helpers

2. **JavaScript Layer** (`assets/salad_ui/`):
   - **State Machines** (`core/state-machine.js`): Handle component behavior and transitions
   - **Component Base Class** (`core/component.js`): Base class providing state management, event handling, ARIA support
   - **Component Implementations** (`components/*.js`): Specific component behavior (dialog, select, popover, etc.)
   - **LiveView Hook** (`core/hook.js`): Bridges Phoenix LiveView and JavaScript components via `SaladUIHook`
   - **Factory Pattern** (`core/factory.js`): Registry system to instantiate components dynamically

3. **Integration Layer**:
   - `lib/salad_ui/liveview.ex`: Server-to-client communication via `SaladUI.LiveView.send_command/4`
   - `lib/salad_ui/liveview.ex`: Client-side JS commands via `SaladUI.JS.dispatch_command/3`
   - Components use `data-component`, `data-part`, `data-action` attributes for JS binding
   - Event mappings defined via `data-event-mappings` attribute (JSON encoded)

### Component Communication

Components communicate through multiple patterns:

- **Server → Client**: Use `SaladUI.LiveView.send_command(socket, component_id, command, params)` to control components from LiveView
- **Client → Client**: Use `SaladUI.JS.dispatch_command(js, command, to: "#component-id")` for JS-based control
- **Client → Server**: Components fire LiveView events via `on-open`, `on-close`, etc. attributes that accept event names or JS commands
- **State Machines**: Components implement transitions that can be triggered by commands (e.g., "open", "close", "toggle")

### Key Patterns

1. **Component Data Attributes**:
   - `data-component="dialog"`: Identifies component type for JS binding
   - `data-part="trigger"`: Identifies sub-elements within a component
   - `data-action="open"`: Defines action triggered by interaction
   - `data-options`: JSON-encoded configuration
   - `data-event-mappings`: JSON-encoded event handlers
   - `phx-hook="SaladUI"`: Attaches the LiveView hook

2. **Variant System**:
   - Defined in component modules or `lib/salad_ui/helpers.ex`
   - Use `variant_class/2` for flexible variant configuration
   - Pre-built `button_variant/1` for button components
   - Variants support size, style, and other visual variations

3. **Form Integration**:
   - `prepare_assign/1` extracts field data from `Phoenix.HTML.FormField`
   - `field_errors/1` and `has_error?/1` for validation state
   - Error translation via configured `:error_translator_function` in config

4. **Dynamic Rendering**:
   - `dynamic/1` component renders tags dynamically based on `tag` attribute
   - `as_child/1` implements shadcn/ui's `asChild` pattern for composition

## File Organization

```
lib/salad_ui/
├── *.ex                    # Individual component modules (button.ex, dialog.ex, etc.)
├── helpers.ex              # Shared helpers (variants, form utils, dynamic rendering)
├── liveview.ex             # LiveView integration (send_command, JS helpers)
└── mix/tasks/              # Mix tasks for installation

assets/salad_ui/
├── index.js                # Entry point, exports SaladUI object
├── core/
│   ├── component.js        # Base Component class
│   ├── state-machine.js    # State machine implementation
│   ├── hook.js             # SaladUIHook for LiveView integration
│   ├── factory.js          # Component registry and factory
│   └── utils.js            # DOM utilities, animations
└── components/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluzky/salad_ui](https://github.com/bluzky/salad_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
