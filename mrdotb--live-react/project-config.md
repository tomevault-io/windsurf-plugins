---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LiveReact is a library that integrates React components into Phoenix LiveView applications with end-to-end reactivity. It combines server-side rendering (SSR), client-side hydration, and bidirectional communication between LiveView and React components.

## Development Commands

### Library Development (Root)

```bash
# Install dependencies
mix deps.get
npm install

# Run tests
mix test

# Format code
mix format
npm run format

# Code quality checks
mix credo

# Compile with warnings as errors
mix compile --force --warnings-as-errors

# Check for unused dependencies
mix deps.unlock --check-unused

# Generate documentation
mix docs
```

### Examples Application (live_react_examples/)

```bash
cd live_react_examples

# Setup (install deps and build assets)
mix setup

# Run the development server
mix phx.server

# Build assets
mix assets.build

# Build assets for deployment
mix assets.deploy

# Install assets dependencies only
mix assets.setup
# Equivalent to: cd assets && npm install

# Asset commands (from assets/ directory)
cd assets
npm run dev          # Start Vite dev server
npm run build        # Build client assets
npm run build-server # Build SSR server bundle
npm run tsc          # Run TypeScript compiler
```

## Architecture

### Core Components

**LiveReact Component (`lib/live_react.ex`)**

- Main Phoenix component that renders React components inside LiveView
- Handles SSR rendering, props serialization, and slot interoperability
- Uses `phx-update="ignore"` to prevent LiveView from overwriting React-managed DOM
- Implements ID generation to avoid collisions while maintaining consistency across dead/live renders

**SSR System (`lib/live_react/ssr.ex`, `lib/live_react/ssr/*.ex`)**

- Behavior-based SSR abstraction with telemetry support
- Two implementations:
  - `ViteJS`: Makes POST requests to Vite dev server during development (requires `config :live_react, vite_host: "http://localhost:5173"`)
  - `NodeJS`: Uses NodeJS supervisor for production rendering (requires `{:nodejs, "~> 3.1"}` dependency)
- Server responses can include preload links by splitting on `<!-- preload -->` marker

**React Hook (`assets/js/live_react/hooks.js`)**

- Phoenix LiveView hook that manages React component lifecycle
- Handles hydration for SSR components vs fresh mounting
- Provides LiveView interop functions via props: `pushEvent`, `pushEventTo`, `handleEvent`, `removeHandleEvent`, `upload`, `uploadTo`
- Manages component unmounting on LiveView navigation

**Vite Plugin (`assets/js/live_react/vite-plugin.js`)**

- Custom Vite plugin that provides `/ssr_render` POST endpoint during development
- Handles hot module reloading for .ex and .heex files
- Configures process termination when Phoenix quits

**Slots System (`lib/live_react/slots.ex`)**

- Converts Phoenix slots to React children
- Only supports default slot (`:inner_block`), passed as React children
- Base64 encodes rendered HTML for transport to client

### Data Flow

1. **Server-side (First Render)**:

   - LiveView calls `<.react name="ComponentName" props... />`
   - SSR renders component if configured and it's a dead view
   - HTML sent to browser with data attributes: `data-name`, `data-props`, `data-slots`, `data-ssr`

2. **Client-side (Mounting)**:

   - React hook reads data attributes from DOM
   - If `data-ssr` present, uses `ReactDOM.hydrateRoot()`; otherwise `ReactDOM.createRoot()`
   - Props include both user-provided props and LiveView interop functions

3. **Updates**:
   - LiveView updates trigger `updated()` hook callback
   - React re-renders with new props
   - React components can call `pushEvent()` to send events back to LiveView
   - LiveView handles events and updates socket assigns

### Directory Structure

```
lib/
  live_react.ex              # Main component
  live_react/
    ssr.ex                   # SSR behavior
    ssr/node_js.ex          # Production SSR via NodeJS
    ssr/vite_js.ex          # Development SSR via Vite
    slots.ex                 # Slot/children handling
    test.ex                  # Test helpers
    reload.ex                # Development reload utilities
  mix/tasks/setup.ex         # Setup task

assets/
  js/live_react/
    index.mjs               # Client-side entry point
    hooks.js                # LiveView hook implementation
    server.mjs              # SSR server entry point
    vite-plugin.js          # Vite plugin for SSR endpoint
    utils.js                # Utilities
  copy/                     # Files copied during installation

live_react_examples/        # Example Phoenix application
  assets/
    react-components/       # Example React components
    js/
      app.js               # Main application entry
      server.js            # SSR server setup
  lib/
    live_react_examples_web/
      live/                # LiveView examples
```

## Key Patterns

### Component Communication

React components receive LiveView interop functions as props:

- `pushEvent(event, payload)` - Send event to current LiveView
- `pushEventTo(selector, event, payload)` - Send event to specific LiveView

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrdotb/live_react](https://github.com/mrdotb/live_react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
