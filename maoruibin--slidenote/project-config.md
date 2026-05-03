---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Si notes, always by your side**
> **侧边笔记，常伴左右**

## Project Overview

SlideNote is a Chrome sidebar extension for quickly storing and accessing fragments of information. It uses the Chrome Side Panel API to live in the browser sidebar **without blocking page content**. Data syncs automatically across devices via Chrome Storage Sync API.

**Product Positioning**: SlideNote is **not a note-taking app**. Think of it as a "sticky note in your browser sidebar" or a "cloud clipboard for fragments" — for quickly storing and retrieving small pieces of information (API keys, server addresses, commands, prompts, etc.).

**Core Value** (in order of importance):
1. **Sidebar, never blocks content** — This is the unique selling point
2. **Instant access** — Opens in < 100ms, no app switching
3. **Auto-sync across devices** — Your fragments follow you everywhere
4. **Real-time search** — Find anything instantly
5. **Markdown support** — Basic formatting for keys, commands, and code (v0.0.3)

**Current Status**: v0.0.7 in development.

## Development Commands

```bash
# Install dependencies
npm install

# Development server (for HTML prototype debugging)
npm run dev

# Build for development (shows "SlideNote Dev" in browser)
npm run build:dev

# Build for production (shows "SlideNote" in browser)
npm run build:prod

# Type checking (JSDoc + TypeScript)
npm run type-check

# Linting
npm run lint
```

### Build Commands Explained

| Command | Display Name | Use Case |
|---------|--------------|----------|
| `npm run build:dev` | SlideNote Dev | Development/testing - can coexist with production version |
| `npm run build:prod` | SlideNote | Production release - for Chrome Web Store and distribution |

## Architecture Overview

### Technology Stack Rationale

The project intentionally avoids frameworks (React/Vue) for:
- **Performance**: Faster load times without ~100KB+ framework overhead
- **Simplicity**: No complex state management needed for basic CRUD
- **Stability**: No framework upgrade risks, code remains viable long-term
- **Control**: Full control over rendering and debugging

### Core Architecture (Planned)

```
src/sidepanel/
├── index.html          # Entry point
├── app.js              # Application initialization
├── styles.css          # Global styles
│
├── core/               # Data layer
│   ├── Store.js        # Chrome Storage API wrapper, state management
│   ├── EventBus.js     # Component communication
│   ├── AutoSaver.js    # Debounced auto-save (1s delay)
│   └── SyncManager.js  # Cross-device sync listener
│
├── components/         # UI components (vanilla JS)
│   ├── Component.js    # Base component class with lifecycle
│   ├── NoteList.js     # Left sidebar note list
│   ├── NoteEditor.js   # Right content editor
│   ├── SearchBar.js    # Search with real-time filtering
│   └── ConfirmDialog.js # Delete confirmation with countdown
│
└── utils/
    ├── dom.js          # DOM helpers
    ├── debounce.js     # Debounce/throttle
    └── icons.js        # Inline SVG icons
```

### Data Flow

1. **User Input** → Component emits event via EventBus
2. **EventBus** → Store methods (createNote, updateNote, deleteNote)
3. **Store** → Chrome Storage Sync API (persists data)
4. **Chrome Storage** → SyncManager detects changes on other devices
5. **SyncManager** → Reloads state, notifies components to refresh

### Storage Design

Chrome Storage Sync API is used with these keys:
- `slidenote_notes`: Array of note objects
- `slidenote_active_id`: Currently selected note ID

Note structure:
```javascript
{
  id: "note_${timestamp}_${random}",
  title: "string",
  content: "string",
  createdAt: number (timestamp),
  updatedAt: number (timestamp)
}
```

**Important Storage Limits:**
- Single item: ~8KB
- Total capacity: ~100KB
- Write frequency: ~1/second (rate-limited)

### Component Communication Pattern

Components do not directly reference each other. All communication flows through the EventBus:

```javascript
// Emit
bus.emit('note:select', noteId);
bus.emit('note:create');
bus.emit('note:delete-request', note);
bus.emit('search:change', query);

// Subscribe
const unsubscribe = bus.on('note:select', (id) => { ... });
```

## Chrome Extension Specifics

### Manifest V3 Configuration

- **Permissions**: Only `storage` (minimal permission)
- **Side Panel**: Uses Chrome 114+ Side Panel API for persistent sidebar
- **Entry Point**: `sidepanel.html` (via manifest side_panel.default_path)

### Debugging

1. **Sidebar DevTools**: Right-click sidebar → "Inspect"
2. **Service Worker**: chrome://extensions/ → "service worker" link
3. **Storage Viewer**: DevTools → Application → Storage → Sync Storage
4. **Sync Testing**: Changes sync across devices in ~10 seconds

## Design System

CSS Variables define the entire design token system (see `docs/versions/v0.0.1/ui-design/prototype.html`):

```css
/* Key tokens */
--color-bg-primary: #ffffff
--color-bg-secondary: #f5f5f5
--color-text-primary: #1a1a1a
--color-primary: #0066cc
--font-size-base: 13px
--spacing-md: 12px
--radius-md: 6px
```

Layout: 480px sidebar width, 180px note list, 300px content area.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maoruibin/SlideNote](https://github.com/maoruibin/SlideNote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
