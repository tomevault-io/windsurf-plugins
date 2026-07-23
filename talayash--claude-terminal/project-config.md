---
trigger: always_on
description: **ClaudeTerminal** is a cross-platform desktop application (Windows and macOS) for managing multiple Claude Code CLI terminal instances from a unified interface. Built with Tauri 2.x (Rust backend) and React 18 (TypeScript frontend), it provides tabbed and grid views of parallel Claude Code sessions with PTY-based terminal emulation. The release workflow produces NSIS/MSI installers for Windows and `.dmg`/`.app` bundles for both Apple Silicon and Intel Macs.
---

# CLAUDE.md

## Project Overview

**ClaudeTerminal** is a cross-platform desktop application (Windows and macOS) for managing multiple Claude Code CLI terminal instances from a unified interface. Built with Tauri 2.x (Rust backend) and React 18 (TypeScript frontend), it provides tabbed and grid views of parallel Claude Code sessions with PTY-based terminal emulation. The release workflow produces NSIS/MSI installers for Windows and `.dmg`/`.app` bundles for both Apple Silicon and Intel Macs.

Current version: **1.28.1**

## Tech Stack

- **Desktop framework**: Tauri 2.x
- **Backend**: Rust (edition 2021)
- **Frontend**: React 18 + TypeScript + Vite
- **Terminal emulation**: xterm.js (`@xterm/xterm`) with fit, search, and web-links addons
- **Styling**: Tailwind CSS + Framer Motion. Flat IntelliJ IDEA 2026.1 "New UI"–style design: a 5-step elevation ramp (`--elevation-0..4` CSS vars), `#3574F0` accent, Inter (UI) + JetBrains Mono (code). Supports dark/light/auto theme, user-set accent color, compact/comfortable/spacious density, UI font scale, and a "reduce motion" toggle that follows the OS `prefers-reduced-motion` setting until explicitly overridden. Text tokens target WCAG AA contrast. (Not glassmorphic — translucency is limited to overlay scrims.)
- **State management**: Zustand (persisted via `zustand/middleware/persist`)
- **Database**: SQLite via `rusqlite` (bundled) — stores profiles, workspaces, session history
- **PTY**: `portable-pty` crate for spawning Claude Code processes
- **Notifications**: `notify-rust` crate for native desktop notifications (Windows Toast and macOS NSUserNotification)
- **Auto-updates**: `tauri-plugin-updater` with signed releases from GitHub

## Project Structure

```
src/                          # React frontend
  App.tsx                     # Root component — layout, event listeners, setup wizard gate
  main.tsx                    # React entry point
  index.css                   # Tailwind base styles
  components/
    TitleBar.tsx              # Custom frameless window titlebar
    Sidebar.tsx               # Terminal list sidebar with search
    TerminalTabs.tsx          # Tab bar for switching terminals
    TerminalView.tsx          # xterm.js terminal renderer
    TerminalGrid.tsx          # Multi-terminal grid view (up to 8)
    TerminalSearch.tsx        # Search within terminal output
    NewTerminalModal.tsx      # Create terminal dialog
    ProfileModal.tsx          # Create/edit configuration profiles
    SettingsModal.tsx         # App settings (args, updates, shortcuts)
    HintsPanel.tsx            # Claude Code command hints reference
    SetupWizard.tsx           # First-run setup (Node.js/Claude Code detection)
    AutoUpdater.tsx           # In-app update UI
    WhatsNewModal.tsx         # Post-update release notes popup
  changelog.json              # Structured release notes data for What's New modal
  store/
    terminalStore.ts          # Terminal instances state (Map<id, {config, xterm}>)
    appStore.ts               # UI state (sidebar, grid, modals, settings)
    updaterStore.ts           # Auto-updater state
  hooks/
    useKeyboardShortcuts.ts   # Global keyboard shortcut handler
    useNotification.ts        # Desktop notification hook

src-tauri/                    # Rust backend
  src/
    main.rs                   # Tauri app setup, plugin registration, state init
    terminal.rs               # TerminalManager — PTY lifecycle (create, write, resize, close)
    commands.rs               # Tauri IPC commands (all #[command] handlers)
    config.rs                 # ConfigProfile struct, HintCategory/Hint structs, default hints
    database.rs               # SQLite database (profiles, workspaces, session_history tables)
  tauri.conf.json             # Tauri config (window, bundling, updater, plugins)
  Cargo.toml                  # Rust dependencies
  capabilities/default.json   # Tauri security capabilities

.claude/commands/
  publish.md                  # /publish slash command — full release workflow

.github/workflows/release.yml  # CI: build + publish GitHub releases (tag-triggered)
```

## Architecture

### Backend (Rust)

- `AppState` holds `Arc<Mutex<TerminalManager>>` and `Arc<Mutex<Database>>`, managed by Tauri
- `TerminalManager` uses `portable-pty` to spawn `cmd /C claude [args]` on Windows (or shell on other platforms)
- Each terminal gets a reader thread that forwards PTY output via `mpsc::channel` to a Tokio task, which emits `terminal-output` events to the frontend
- When the PTY reader loop ends (process exit), a `terminal-finished` event is emitted
- All Tauri commands are async and defined in `commands.rs`
- Shell commands (`node`, `npm`, `claude`) are wrapped via `cmd /C` on Windows with `CREATE_NO_WINDOW` flag

### Frontend (React)

- `App.tsx` listens for `terminal-output` and `terminal-finished` Tauri events

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talayash/claude-terminal](https://github.com/talayash/claude-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
