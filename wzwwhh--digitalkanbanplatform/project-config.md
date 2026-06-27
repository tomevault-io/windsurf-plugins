---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-driven data visualization dashboard platform. Users describe requirements in natural language, and AI generates complete dashboards. Supports manual drag-and-drop editing, real-time data binding, and one-click HTML export.

**Core Philosophy**: "Connect data, say a sentence, export and deploy directly"

## Development Commands

### Frontend (Vue 3 + Vite)
```bash
cd frontend
npm install
npm run dev      # → http://localhost:5175
npm run build    # Production build
npm run preview  # Preview production build
```

### Backend (FastAPI + Python)
```bash
cd backend
pip install -r requirements.txt

# Configure AI (optional - bulk operations/theme switching work without it)
echo "MOONSHOT_API_KEY=your_key" > .env

python -m uvicorn app.main:app --port 8000 --reload
```

### Database Initialization (for testing)
```bash
# SQLite sample data (built-in)
cd backend && python init_sample_db.py

# MySQL test data (optional)
mysql -u root -p < doc/test_mysql_init.sql
```

## Architecture

### System Theme vs Board Style Separation

**Critical Design**: The platform separates **system theme** (UI shell) from **board style** (canvas visuals):

- **System Theme** (`applySystemTheme`): Controls app shell (topbar, sidebar, buttons)
- **Board Style** (`applyBoardStyle`): Controls canvas area only (background, component borders, chart colors)
- **AI Boundary**: AI's `CHANGE_THEME` command only affects board style, never system theme

Implementation: `frontend/src/stores/theme.js` (lines 23-62)

### Three-Layer Intent Routing

AI requests flow through three levels to minimize token usage:

```
User Message → Level 1: Regex (0 tokens, exact commands like "删除")
            → Level 2: Keywords (0 tokens, themes/colors/charts)
            → Level 3: LLM Agent (Kimi, scene generation/component modification)
```

**Design Principle**: "Better to miss and send to LLM than to misinterpret and do the wrong thing"

Configuration: `backend/app/ai_config.py`
Router: `backend/app/routers/ai.py`

### Command System

All operations (AI and manual) produce Command objects that are:
- Serializable JSON
- Undoable/redoable
- Uniform interface for both AI and user actions

Command types:
- `ADD_WIDGET`, `UPDATE_WIDGET`, `DELETE_WIDGET`
- `MOVE_WIDGET`, `RESIZE_WIDGET`
- `BATCH` (for multi-component operations)

Implementation: `frontend/src/core/command.js`

### Registry-Driven Architecture

All extensible components use a central registry pattern:

```javascript
// frontend/src/core/registry.js
registerWidget(type, config)   // 15 visualization components
registerTheme(name, config)     // 5 built-in + unlimited custom themes
registerDatasource(type, config) // API, Database adapters
```

To add new components/themes: register in corresponding `_registry.js` files, no core code changes needed.

### Data Model

```javascript
Project {
  id, name, theme,
  dataSources: []  // Project-level, shared across all dashboards
}

Dashboard {
  id, projectId, name,
  widgets: [],     // Each dashboard has its own component list
  createdAt, savedAt
}

Widget {
  id, type,
  props: { title, value, color, ... },  // All properties AI can modify
  position: { x, y },                    // AI can modify ("move to top-right")
  size: { w, h },                        // AI can modify ("make it bigger")
  dataSource: {                          // Component-level data binding (optional)
    sourceId: "ds_1",                    // References project-level datasource
    mapping: {                           // Field mapping
      x: "date",                         // X-axis field
      y: "amount",                       // Y-axis field
      series: "category",                // Series field (optional)
      value: "total"                     // KPI value field
    },
    interval: 30000                      // Auto-refresh interval (ms), 0=disabled
  }
}
```

### Hybrid Workflow

Users can freely mix AI generation and manual editing:

1. Create dashboard → Enter editor
2. Freely do any of these (no fixed order, repeatable):
   - Manually drag components from panel
   - Tell AI "add a KPI card"
   - Go to datasource page and configure an API
   - Return and tell AI "use that API data to make a line chart"
   - Manually adjust position and style
   - Tell AI "pick a few more suitable charts from the data"
   - Continue manual fine-tuning → Export

**Key Principle**: Canvas is always the source of truth. AI and manual operations both **add** to the canvas, never overwrite each other.

### AI Context

Every AI request must include full context:

```javascript
context = {
  widgets: [{ id, type, props, position, size, dataSource }],  // All components on canvas
  selectedId: "widget_xxx" | null,                             // Currently selected component
  dataSources: [{ id, name, type, fields, sample }],           // All available datasources
  theme: "dark-tech"                                           // Current theme
}
```

This enables AI to make informed decisions like:
- "add a chart" → AI sees sales API available, auto-selects line chart and binds data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wzwwhh/digitalKanbanPlatform](https://github.com/wzwwhh/digitalKanbanPlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
