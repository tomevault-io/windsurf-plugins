---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this app is

A macOS Tauri 2 desktop app (Vue 3 + Rust) for browsing, viewing, and trashing
local session transcripts from coding agent CLIs — currently **Claude Code**,
**Codex**, and **Gemini CLI**. Each CLI stores JSONL transcripts in
its own on-disk layout; this app normalizes them all into the same project →
sessions → chat UI, plus a soft-delete trash that survives across agents. The
app is read-only against the original transcripts — deletion is a `move` into a
trash dir, never `rm`.

## Commands

```bash
npm run tauri dev        # full dev (Tauri shell + Vite on :1420)
npm run tauri build      # bundle .app / .dmg into src-tauri/target/release/bundle/
npm run dev              # web-only Vite preview; Tauri invokes will fail
npm run build            # vue-tsc --noEmit + vite build
npm test                 # vitest watch mode
npm run test:run         # vitest single run (CI)
npm run test:coverage    # vitest single run + v8 coverage report
```

There is no linter wired up — `npm run build` (which runs `vue-tsc --noEmit`
first) is the typecheck step.

Unit tests run on **Vitest** (jsdom env) and live under `test/`, mirroring
`src/`. They cover the agent-agnostic logic modules (`format`, `i18n`,
`settings`, `chatToolbar`, `trashToolbar`, `sessionsToolbar`, `export`, `api`,
`fly`, `tooltip`) and the leaf components (`DiffBlock`, `ToolResult`,
`CollapsibleBox`, `Sidebar`, `SidebarTopbar`, `SessionsTopbar`, `TrashTopbar`,
`SettingsModal`). Config is `vitest.config.ts` (separate from
`vite.config.ts`); jsdom polyfills for `matchMedia` / `ResizeObserver` /
`Element.animate` live in `test/setup.ts`. `App.vue`, `views/`, and `modals/`
are stateful shells left to manual/e2e testing and excluded from coverage.
`test/tsconfig.json` is IDE-only — the production build never type-checks
`test/`.

Vite is locked to port `1420` (strictPort) because `tauri.conf.json` hardcodes
that URL. `src-tauri/**` is excluded from Vite's watcher; Rust changes are
picked up by Tauri's own dev loop.

## Architecture

### Two-side split

- **Frontend** (`src/`) is a thin Vue 3 SPA. State lives in `App.vue` refs;
  there is no store. All persistence besides `localStorage` (lang/theme/pin
  prefs) goes through Tauri.
- **Backend** (`src-tauri/src/`) owns *all* filesystem I/O and JSONL parsing.
  Frontend calls it via the `#[tauri::command]` functions in `lib.rs`, wrapped
  by `src/api.ts`. The full handler list lives in `tauri::generate_handler!` at
  the bottom of `lib.rs`; keep it in sync.

The backend is split into:

```
src-tauri/src/
├── lib.rs           // Tauri commands + macOS setup; pure dispatch, no parsing
├── types.rs         // Serializable types shared with the frontend
├── util.rs          // dirs / time / jsonl / text helpers (agent-agnostic)
├── trash.rs         // soft-delete / restore / list / empty (agent-agnostic)
└── agents/
    ├── mod.rs       // `SessionSource` trait + `source(agent)` dispatcher
    ├── claude.rs    // ClaudeSource impl  (~/.claude/projects/<dir>/...)
    └── codex.rs     // CodexSource impl   (~/.codex/sessions/<YYYY>/...)
```

When adding a Tauri command, define it in `lib.rs`, register it in
`tauri::generate_handler!`, then expose it from `api.ts` with the matching
TypeScript types in `src/types.ts`. `serde(rename_all = "camelCase")` is set on
every type in `types.rs` so Rust snake_case fields land in JS as camelCase.

### Session-source abstraction (adding a new agent)

The backend hides each agent's on-disk layout behind a single `SessionSource`
trait defined in `agents/mod.rs`. Currently:

| Agent  | Layout                                                              | Project grouping                |
| ------ | ------------------------------------------------------------------- | ------------------------------- |
| Claude | `~/.claude/projects/<dir>/<sessionId>.jsonl`                        | by project directory            |
| Codex  | `~/.codex/sessions/<YYYY>/<MM>/<DD>/rollout-*.jsonl`                | by the `cwd` recorded *inside* each file |
| Gemini | `~/.gemini/tmp/<slug>/chats/session-*.jsonl` (+ `.project_root` sibling) | by `slug`; cwd read from `.project_root` |

To add a new agent (template: see `agents/gemini.rs`):

1. Create `src-tauri/src/agents/<name>.rs` with a `<Name>Source` unit struct
   that implements `SessionSource` (every method calls the agent's private
   parsing helpers in the same file).
2. Add `pub mod <name>;` and a match arm in `agents::source()`.
3. Add `"<name>"` to the `Agent` union type in `src/types.ts` — sidebar /
   agent-switcher pick it up automatically.

That's it. The Tauri commands (`list_projects`, `list_sessions`,
`read_session`, `rename_session`, `soft_delete_session`, `resume_session`, …)
all dispatch through `agents::source(&agent)?.<method>()`, so no command
plumbing changes. **Do not** add agent-specific match arms in `lib.rs` or
`trash.rs`; if you can't fit a piece of logic on the trait, the trait shape is
wrong — fix it there.

`list_sessions` is paginated; it sorts by mtime cheaply and only deep-parses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerrywu001/cc-sessions-viewer](https://github.com/jerrywu001/cc-sessions-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
