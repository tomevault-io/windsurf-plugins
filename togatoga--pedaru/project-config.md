---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pedaru is a cross-platform desktop PDF viewer built with Tauri 2.x and React/Next.js. It provides advanced features like tab management, standalone windows, full-text search, bookmarks, and navigation history with persistent session storage.

## Development Commands

### Prerequisites

- Node.js >= 18.17.0
- Rust >= 1.85
- Tauri CLI

### Setup

```bash
# Development
npm install                      # Install dependencies
npm run tauri dev                # Run app with hot reload
npm run tauri dev -- -- /path    # Open specific PDF file

# Build
npm run build                    # Build Next.js frontend
npm run tauri build              # Create platform bundles (.dmg, .deb, .msi)
npm run tauri build -- --debug   # Build with debug symbols

# Testing
cargo test --verbose             # Run Rust tests (in src-tauri/)
npm test                         # Run frontend unit tests (Vitest)
npm run test:ui                  # Run tests with Vitest UI
npm run test:coverage            # Run tests with coverage report
npm run typecheck                # TypeScript type checking
cargo clippy -- -D warnings      # Rust linting (in src-tauri/)
cargo fmt -- --check             # Rust formatting check (in src-tauri/)

# Linting & Formatting (Frontend)
npm run lint                     # Run Biome linter
npm run lint:fix                 # Run Biome linter with auto-fix
npm run format                   # Format code with Biome/Prettier
npm run format:check             # Check formatting without changes
```

### Debugging

#### Testing PDF File Opening

To test opening a PDF file via command line (simulating double-click behavior):

```bash
# Use absolute path (relative paths won't work)
npm run tauri dev -- -- /absolute/path/to/file.pdf

# Example
npm run tauri dev -- -- /Users/username/Documents/sample.pdf
```

To test without opening a PDF (restores last opened file):

```bash
npm run tauri dev
```

#### Viewing Logs

**Development mode (`npm run tauri dev`):**
- Rust logs (`eprintln!`) appear in the terminal
- Frontend logs (`console.log`) appear in the WebView DevTools (right-click → Inspect Element)

**Production build:**

```bash
# Build with debug symbols
npm run tauri build -- --debug

# Run the app and view logs in Console.app (macOS)
# Filter by "Pedaru" to see app-specific logs
open /Applications/Utilities/Console.app
```

Or run the built app from terminal to see logs:

```bash
# After building
./src-tauri/target/release/bundle/macos/Pedaru.app/Contents/MacOS/Pedaru
```

#### Testing File Associations (macOS)

File associations only work with the built app:

```bash
# Build the app
npm run tauri build

# The app is created at:
# src-tauri/target/release/bundle/macos/Pedaru.app

# Test by:
# 1. Right-click a PDF in Finder → Open With → Pedaru
# 2. Or drag a PDF onto Pedaru.app icon
# 3. Or double-click a PDF after setting Pedaru as default PDF app
```

**Test Framework:**
- Frontend tests use Vitest with jsdom environment
- Test files are colocated next to source files: `src/lib/*.test.ts` and `src/hooks/*.test.ts`
- Coverage includes all files in `src/lib/` and `src/hooks/`

## Architecture

### Frontend-Backend Communication

The app uses Tauri's IPC system for frontend-backend communication:

**Rust Commands (src-tauri/src/lib.rs):**
- `get_pdf_info(path)` - Extracts metadata, TOC, author info with multi-encoding support (UTF-8, UTF-16BE, Shift-JIS, EUC-JP, ISO-2022-JP)
- `read_pdf_file(path)` - Returns raw PDF bytes for rendering
- `get_opened_file()` - Retrieves CLI-provided or "Open With" file path
- `was_opened_via_event()` - Checks if app was launched by opening a file

**Frontend Invocation (TypeScript):**
```typescript
import { invoke } from '@tauri-apps/api/core';
const pdfInfo = await invoke<PdfInfo>('get_pdf_info', { path });
```

### Multi-Window Architecture

The app supports two window types:

1. **Main Window** - Primary viewer with tabs, sidebars, search, and full controls
2. **Standalone Windows** - Independent page viewers (like macOS Preview)

**Window Coordination** via Tauri events:
- `window-page-changed` - Notifies when a window navigates to a different page
- `window-state-changed` - Syncs zoom level and view mode changes
- `bookmark-sync` - Synchronizes bookmarks across all windows
- `move-window-to-tab` - Converts standalone window back to tab in main window

When editing multi-window features, ensure events are emitted and handled properly in both main and standalone windows.

### Session Persistence

Session state is stored in SQLite database with per-PDF granularity:

**Database Location:**
- macOS: `~/Library/Application Support/pedaru/pedaru.db`
- Linux: `~/.config/pedaru/pedaru.db`
- Windows: `C:\Users\<username>\AppData\Roaming\pedaru\pedaru.db`

**Database Schema:**
- `sessions` table - Per-PDF session data:
  - Current page, zoom level, view mode
  - Open tabs and their states (JSON)
  - Standalone windows configuration (JSON)
  - Bookmarks (JSON array of {page, label, timestamp})
  - Navigation history (JSON array, max 100 entries)
  - File path and path hash for quick lookup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [togatoga/pedaru](https://github.com/togatoga/pedaru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
