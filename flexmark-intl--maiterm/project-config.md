---
trigger: always_on
description: A Tauri-based terminal emulator with workspace organization, built with Svelte 5 and Rust.
---

# maiTerm

A Tauri-based terminal emulator with workspace organization, built with Svelte 5 and Rust.

## Tech Stack

- **Frontend**: Svelte 5 (runes), SvelteKit, TypeScript, Vite
- **Backend**: Rust, Tauri 2
- **Terminal**: alacritty_terminal (Rust VTE parser + buffer) with xterm.js as thin renderer (scrollback=0; DOM renderer by default — canvas/webgl ghost under full-frame streaming; fit, web-links addons)
- **Editor**: CodeMirror 6 (+ MergeView for diffs)
- **PTY**: portable-pty for cross-platform pseudo-terminal support
- **State**: parking_lot RwLock for thread-safe Rust state

## Project Structure

```
src/                          # Frontend (Svelte/TypeScript)
├── routes/                   # SvelteKit routes
│   ├── +layout.svelte        # App shell, keyboard shortcuts, modals
│   └── +page.svelte          # Main terminal view, portal rendering
├── lib/
│   ├── components/           # Svelte components
│   │   ├── editor/           # EditorPane (CodeMirror 6), DiffPane (MergeView)
│   │   ├── terminal/         # TerminalPane, TerminalTabs
│   │   ├── workspace/        # WorkspaceSidebar
│   │   └── pane/             # SplitPane
│   ├── stores/               # Svelte 5 stores (.svelte.ts)
│   │   ├── workspaces.svelte.ts   # Workspace/pane/tab CRUD, navigateToTab()
│   │   ├── terminals.svelte.ts    # Terminal instances, OSC state
│   │   ├── preferences.svelte.ts  # User preferences
│   │   ├── activity.svelte.ts     # Tab activity indicators (OSC 133)
│   │   ├── triggers.svelte.ts     # Trigger engine (pattern matching, variables)
│   │   ├── claudeCode.svelte.ts   # Claude Code IDE tool request handler
│   │   ├── claudeState.svelte.ts  # Claude session state from hooks (active/idle/permission)
│   │   ├── sshMcpBridge.svelte.ts # SSH MCP bridge orchestration, reactive status
│   │   ├── editorRegistry.svelte.ts # Editor state tracking (dirty, view refs)
│   │   ├── notifications.svelte.ts  # Command completion notification logic
│   │   ├── toasts.svelte.ts       # In-app toast notification store
│   │   └── notificationDispatch.ts # Routes to toast or OS notification
│   ├── triggers/             # Trigger definitions and parsing
│   ├── themes/               # Theme system
│   ├── utils/                # Pure utility modules
│   └── tauri/                # Tauri IPC layer (commands.ts, types.ts)

src-tauri/src/                # Backend (Rust)
├── lib.rs                    # Tauri app setup, command registration
├── commands/                 # Tauri command handlers
├── claude_code/              # Claude Code IDE integration (MCP server)
├── comms/                    # Comms integration (/maiterm resolve): Mattermost client + thread-reply watcher
├── terminal/                 # Terminal backend (alacritty_terminal)
│   ├── handle.rs             # TerminalHandle, TermDimensions, create_terminal()
│   ├── event_proxy.rs        # AitermEventProxy (EventListener → Tauri events)
│   ├── render.rs             # Grid → ANSI viewport renderer (~60fps)
│   ├── osc.rs                # OscInterceptor (OSC 7/9/133/633/1337)
│   ├── search.rs             # Buffer search via RegexSearch
│   └── serialize.rs          # Buffer serialization/restore via VTE parser
├── state/                    # Application state + persistence
└── pty/                      # PTY management
```

**Module-specific docs**: Detailed documentation for individual subsystems lives in CLAUDE.md files within their directories:
- `src/lib/components/terminal/CLAUDE.md` — Portal pattern, terminal architecture (alacritty_terminal + xterm.js), OSC, shell integration, split cloning
- `src/lib/components/editor/CLAUDE.md` — CodeMirror, diff tabs, editor registry
- `src-tauri/src/claude_code/CLAUDE.md` — Claude Code IDE integration, SSH MCP bridge
- `src/lib/triggers/CLAUDE.md` — Trigger engine, defaults, variables, dedup

## Commands

```bash
npm run dev          # Start Vite dev server (frontend only)
npm run check        # TypeScript + Svelte type checking
npm run tauri:dev    # Full app development (frontend + backend + MCP bridge)
npm run tauri:build  # Production build
cargo check          # Check Rust compilation (in src-tauri/)
```

**Note**: `npm run tauri:dev` passes `--features mcp-bridge --config src-tauri/tauri.dev.conf.json` to enable the Claude Code MCP bridge and apply dev-specific CSP overrides.

## Key Patterns

### Svelte 5 Stores

Stores use the runes API with a factory function pattern:

```typescript
function createMyStore() {
  let value = $state<Type>(initial);

  return {
    get value() { return value; },  // Getter for reactivity

    async setValue(newValue: Type) {
      await commands.setValue(newValue);  // Persist to backend
      value = newValue;                   // Update local state
    }
  };
}

export const myStore = createMyStore();
```

### Tauri Commands

1. Define Rust struct in `state/workspace.rs` with serde derives
2. Add command in `commands/workspace.rs`
3. Register in `lib.rs` invoke_handler
4. Add TypeScript type in `tauri/types.ts`
5. Add wrapper function in `tauri/commands.ts`

### Component Patterns

- **Modals**: Follow `HelpModal.svelte` pattern (backdrop, escape key, close button)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flexmark-Intl/maiterm](https://github.com/Flexmark-Intl/maiterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
