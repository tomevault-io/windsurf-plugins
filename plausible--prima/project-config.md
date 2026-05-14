---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prima is a Phoenix LiveView component library providing unstyled, accessible UI components. The repository is structured as:
- **Root directory** - Library code (published to Hex)
- **demo/ directory** - Demo application for development and testing

## Essential Commands

### Library Development (from root)
```bash
mix deps.get            # Get library dependencies
mix compile             # Compile library
mix assets.build        # Build library JavaScript (prima.js)
mix hex.build           # Build hex package for publishing
```

### Demo Application (from demo/)
```bash
cd demo
mix setup               # Full setup (deps, assets setup, assets build)
mix deps.get            # Get demo dependencies
mix assets.setup        # Install Tailwind and esbuild
mix assets.build        # Build demo assets

# Check if server is already running at localhost:4000 before starting
mix phx.server          # Start development server (demo app at localhost:4000)
```

### Testing

**Library tests (from root):**
```bash
mix test                    # Run library unit tests
```

**Demo/integration tests (from demo/):**
```bash
cd demo
mix test                    # Run all tests including Wallaby browser tests
mix test path/to/file       # Run tests for a single file
mix test path/to/file:123   # Run a single test starting on given line
```

For Wallaby browser tests specifically:
- Tests are in `demo/test/wallaby/demo_web/`
- ChromeDriver-based integration tests for UI interactions
- Run with standard `mix test` command from demo/ directory

## Architecture & Component Patterns

### Repository Structure
```
prima/                       # Root - library code only
  lib/prima/                 # Library components (published to Hex)
    modal.ex
    dropdown.ex
    combobox.ex
  assets/js/                 # Library JavaScript
    prima.js                 # Main export
    hooks/                   # Component hooks
  priv/static/assets/        # Built library bundle
    prima.js
  mix.exs                    # Library-only dependencies
  config/config.exs          # Library build config (esbuild only)

  demo/                      # Demo app (not published)
    lib/demo/
      application.ex         # Demo OTP application
    lib/demo_web/            # Demo Phoenix application
      endpoint.ex
      router.ex
      live/                  # Demo pages and fixtures
    config/                  # Demo config files
    assets/                  # Demo assets
    test/                    # Demo tests including Wallaby
    mix.exs                  # Demo dependencies (includes {:prima, path: ".."})
```

### Core Architecture
The library follows a three-layer pattern for each component:
1. **Phoenix Component** (`lib/prima/*.ex`) - Server-side rendering and LiveView integration
2. **JavaScript Hook** (`assets/js/hooks/*.js`) - Client-side behavior and DOM manipulation
3. **CSS Integration** - Tailwind-based styling with standard data attribute selectors

### Component Structure
```elixir
# Standard component pattern
defmodule Prima.ComponentName do
  use Phoenix.Component

  # Main component function with slots and attributes
  attr :id, :string, required: true
  slot :inner_block, required: true
  def component_name(assigns) do
    ~H"""
    <div phx-hook="ComponentName" prima-ref={@id}>
      <%= render_slot(@inner_block) %>
    </div>
    """
  end
end
```

### Custom Data Attributes
- `data-prima-ref` - Component instance identifier
- `data-focus` - Focus state for dropdown items (true/false)
- Standard data attributes for component state management

### Current Components
- **Modal** - Dialog/popup with overlay (`lib/prima/modal.ex`)
- **Dropdown** - Menu/select functionality (`lib/prima/dropdown.ex`)
- **Combobox** - Searchable input with suggestions (`lib/prima/combobox.ex`)

## Development Workflow

### Browser-Based Exploration and Debugging

**Playwright MCP is the preferred tool for exploring and debugging the demo application.**

- The development server runs on `http://localhost:4000`
- Start the server from demo/ directory: `cd demo && mix phx.server`
- **Always assume the server is running** - navigate to `localhost:4000` to view components
- If nothing loads or the server isn't running, ask the user how to continue
- Use Playwright MCP tools to:
  - Navigate to demo pages and view components visually
  - Take screenshots to understand current UI state
  - Interact with components (click, hover, type) to test behavior
  - Inspect page snapshots to understand DOM structure
- Prefer Playwright MCP to using curl

**Debugging JavaScript Hooks:**
- Add `console.log()` statements in JavaScript hook files (`assets/js/hooks/*.js`)
- Use Playwright's browser console tools to view console output
- Console logs are very useful for understanding hook behavior and state changes
- Consider adding temporary debug logs when troubleshooting component issues

### Testing Strategy
- **ExUnit tests** for simple component logic and rendering (library tests in root `test/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plausible/prima](https://github.com/plausible/prima) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
