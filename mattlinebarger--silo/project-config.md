---
trigger: always_on
description: Silo is an Electron-based macOS desktop wrapper for Google Workspace (Gmail, Calendar, Drive, Keep, Tasks, Contacts) and Gemini. It uses **BrowserViews** (not BrowserWindows or webview tags) to create persistent, side-by-side views that feel native.
---

# Silo AI Coding Instructions

## Project Overview

Silo is an Electron-based macOS desktop wrapper for Google Workspace (Gmail, Calendar, Drive, Keep, Tasks, Contacts) and Gemini. It uses **BrowserViews** (not BrowserWindows or webview tags) to create persistent, side-by-side views that feel native.

**Core Architecture**: Single main window + sidebar BrowserView + multiple content BrowserViews (one per Google app). Only one content view is visible at a time alongside the persistent sidebar.

## Key Architectural Patterns

### BrowserView Management ([src/main/main.js](src/main/main.js))

- **BrowserViews over tabs**: Each Google app lives in its own `BrowserView`, not a tab or separate window. Views persist in memory even when hidden, maintaining state.
- **View switching**: `showView(name)` removes all views from the window, then re-adds sidebar + the target content view. Uses `setBounds()` for layout, not CSS.
- **Fixed sidebar**: The sidebar is always 60px wide, positioned at x:0. Content views start at x:60 and fill remaining width.
- **View registry**: All views stored in the `views` object keyed by name (mail, calendar, drive, etc.). The `currentView` global tracks which content view is active.

### Security & URL Handling

- **Internal domain whitelist**: `INTERNAL_DOMAINS` array defines allowed Google domains. Any external link opens in the default browser via `shell.openExternal()`.
- **Window open handler**: `setWindowOpenHandler()` checks `isInternalUrl()` for all new window requests. Only internal URLs get `{ action: "allow" }`.
- **Navigation guard**: `will-navigate` event prevents navigation to external domains and redirects them externally.

### IPC Communication

- **Preload scripts segregated**:
  - [preload.js](src/preload/preload.js) for content views - exposes `notify()` and `unreadCount()` to Gmail
  - [sidebar-preload.js](src/preload/sidebar-preload.js) for sidebar - exposes `switch()` and `onActiveChange()`
- **Badge updates**: Gmail view sends `unread-count` IPC message, main process updates macOS dock badge
  - Implementation may use Google's existing APIs or DOM scraping when APIs unavailable
  - Check Gmail's web interface for exposed notification data before implementing custom scraping
- **View switching**: Sidebar clicks trigger `sidebar-switch` IPC, main process calls `showView()`

### Menu & Keyboard Shortcuts

- **Cmd+1 through Cmd+7**: Switch between the 7 Google apps
- **Cmd+N**: Compose new email (opens in separate `BrowserWindow`)
- **Cmd+,**: Open settings view
- **Template icons**: Menu icons use `nativeImage.setTemplateImage(true)` for native macOS appearance

## Development Workflow

### Running the App

```bash
npm start              # Default: Gmail wrapper
npm run start:gmail    # Explicit Gmail
npm run start:calendar # Start with Calendar active
```

The `WRAPPER_APP` environment variable determines the initial view, but this is legacy - the app now shows all services simultaneously via sidebar switching.

### Building for macOS

```bash
npm run build
```

Uses `electron-builder` with custom `build` config in [package.json](package.json). Output goes to `dist/`. Currently configured for Mac only with:
- `public.app-category.productivity` category
- `mailto:` protocol handler registration
- Code signing and notarization disabled (`sign: false`, `notarize: false`)

### Project Structure

```
src/
  main/main.js          # Main process - window/view management, menus, IPC
  preload/
    preload.js          # Preload for content views (Google apps)
    sidebar-preload.js  # Preload for sidebar
  renderer/
    sidebar.html        # 60px sidebar UI with Material icons + custom SVGs
    settings.html       # Settings view (currently placeholder)
    img/                # Custom icons (drive.png, gemini.png) as CSS masks
assets/
  icon.icns            # App icon
  menu/*.png           # Template images for native menus
```

## Code Style & Conventions

- **No frameworks**: Vanilla JavaScript, plain HTML/CSS. No React, Vue, or similar.
- **CommonJS**: Uses `require()`, not ES modules. `"type": "commonjs"` in package.json.
- **Inline styles**: Renderer HTML files have `<style>` blocks, no separate CSS files.
- **Material Symbols**: Sidebar uses Google's Material Symbols font (outlined style). Custom icons (Drive, Gemini) are PNG masks applied via CSS.
- **Template literals for file paths**: `file://${path.join(__dirname, "../renderer/settings.html")}`

## Testing Approach

This is a solo project with **manual testing only**. No automated test suite exists. When making changes:
- Test all 7 Google app views (mail, calendar, drive, gemini, keep, tasks, contacts)
- Verify keyboard shortcuts (Cmd+1-7, Cmd+N, Cmd+,, Cmd+R)
- Check both light and dark mode appearance
- Test external link handling (should open in default browser)
- Verify sidebar active state syncs correctly with view switching

## Custom Icon Usage

Drive and Gemini icons are custom PNGs in [src/renderer/img/](src/renderer/img/). Applied as CSS masks:

```css
.drive-icon {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattlinebarger/silo](https://github.com/mattlinebarger/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
