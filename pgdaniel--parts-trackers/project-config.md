---
trigger: always_on
description: This is a **Rails 8.1** application using the modern Rails stack with SQLite for all storage needs (database, cache, job queue, and Action Cable). The project uses a **unique hybrid architecture**: each Rails view renders a **complete standalone SPA** with **ActionCable for real-time data transfer**.
---

# Copilot Instructions for Parts Tracker

## Architecture Overview

This is a **Rails 8.1** application using the modern Rails stack with SQLite for all storage needs (database, cache, job queue, and Action Cable). The project uses a **unique hybrid architecture**: each Rails view renders a **complete standalone SPA** with **ActionCable for real-time data transfer**.

Key architectural decisions:
- **One Rails route → One complete SPA**: Each major section (e.g., `/tools`, `/inventory`) is a full React application
- **React Router** handles all routing within each SPA (no page reloads)
- **Zustand** for state management within each SPA
- **ActionCable for data**: Use WebSocket channels instead of REST/JSON APIs for all client-server communication
- **Stimulus mounts React**: Each SPA is mounted via a Stimulus controller for clean lifecycle management
- **Solid Queue** for background jobs (runs in-process with Puma via `SOLID_QUEUE_IN_PUMA=true`)
- **Solid Cache** and **Solid Cable** for performance and real-time features (Solid Cable backs ActionCable)
- **Propshaft** for asset pipeline (not Sprockets)
- **esbuild** for JavaScript bundling (via jsbundling-rails)
- **SQLite multi-database setup**: separate databases for primary, cache, queue, and cable in production
- **Kamal** deployment configured for containerized deployment

## Development Workflows

### Starting the Application
```bash
bin/dev  # Runs Procfile.dev: Rails server + esbuild watch mode
```
This starts two processes: the Rails server with Ruby debugger enabled and JavaScript building in watch mode.

### Setup/Reset
```bash
bin/setup              # Initial setup: bundle, yarn, db:prepare
bin/setup --reset      # Full reset: drops and recreates database
```

### Running Tests
```bash
bin/rails test         # Run all tests (parallelized by default)
bin/rails test:system  # System tests with Selenium/Chrome headless
bin/ci                 # Full CI suite: tests + linting + security audits
```

### Code Quality & Security
```bash
bin/rubocop           # Ruby style (follows Rails Omakase styling)
bin/brakeman          # Security vulnerability scanner
bin/bundler-audit     # Check for vulnerable gem versions
```

## JavaScript/Frontend Conventions

Each Rails route renders a **complete standalone SPA** that manages all its own routing, state, and UI. Think of Rails as providing the entry point and data layer, while React takes over the entire user experience within that section.

### Structure
- **Complete SPAs** in `app/javascript/apps/` (e.g., `ToolsApp.jsx`, `InventoryApp.jsx`)
- **Stimulus controllers** in `app/javascript/controllers/` - mount React apps with lifecycle management
- **React components** in `app/javascript/components/` - organized by feature/domain
- **Zustand stores** in `app/javascript/stores/` - state management per app
- **ActionCable channels** in `app/javascript/channels/` for real-time data
- Entry point: `app/javascript/application.js`
- Build output: `app/assets/builds/application.js` (generated, don't edit)

### Complete SPA Pattern
A single Rails route (e.g., `/tools`) loads a full React application that handles all sub-routes internally:

```javascript
// app/javascript/apps/ToolsApp.jsx
import React from 'react'
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { useToolsStore } from '../stores/toolsStore'
import ToolsList from '../components/tools/ToolsList'
import ToolDetail from '../components/tools/ToolDetail'
import ToolEdit from '../components/tools/ToolEdit'

export default function ToolsApp({ channel }) {
  return (
    <BrowserRouter basename="/tools">
      <Routes>
        <Route path="/" element={<ToolsList channel={channel} />} />
        <Route path="/:id" element={<ToolDetail channel={channel} />} />
        <Route path="/:id/edit" element={<ToolEdit channel={channel} />} />
        <Route path="*" element={<Navigate to="/" replace />} />
      </Routes>
    </BrowserRouter>
  )
}
```

```javascript
// app/javascript/stores/toolsStore.js
import { create } from 'zustand'

export const useToolsStore = create((set) => ({
  tools: [],
  currentTool: null,
  setTools: (tools) => set({ tools }),
  setCurrentTool: (tool) => set({ currentTool: tool }),
  updateTool: (id, updates) => set((state) => ({
    tools: state.tools.map(t => t.id === id ? { ...t, ...updates } : t),
    currentTool: state.currentTool?.id === id ? { ...state.currentTool, ...updates } : state.currentTool
  }))
}))
```

```javascript
// app/javascript/controllers/tools_app_controller.js
import { Controller } from "@hotwired/stimulus"
import React from 'react'
import { createRoot } from 'react-dom/client'
import ToolsApp from '../apps/ToolsApp'
import consumer from '../channels/consumer'

export default class extends Controller {
  connect() {
    this.root = createRoot(this.element)
    this.setupChannel()
  }
  
  disconnect() {
    if (this.channel) {
      this.channel.unsubscribe()
    }
    if (this.root) {
      this.root.unmount()
    }
  }
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgdaniel/parts_trackers](https://github.com/pgdaniel/parts_trackers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
