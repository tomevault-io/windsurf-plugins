---
trigger: always_on
description: Cross-platform terminal with split panes, tab groups, workspaces, a CodeMirror
---

# TEDI

Cross-platform terminal with split panes, tab groups, workspaces, a CodeMirror
editor, a BYOK AI agent and a runtime extension system. Tauri 2 + Rust owns every
OS resource; a React 19 webview owns the UI. Forked from
[Crynta/Terax v0.5.9](https://github.com/crynta/terax-ai).

Stack: Tauri 2, Rust (`portable-pty`, `russh`), React 19 + TS, xterm.js (WebGL),
CodeMirror 6, shadcn/ui, Tailwind v4, `@ai-sdk/*` v6. Package manager is **pnpm**.

## Commands

```bash
pnpm exec tsc --noEmit    # frontend types
pnpm lint:imports         # module import discipline
pnpm verify               # the invariant suite; `pnpm verify ai` filters to one folder
pnpm build                # frontend build
pnpm tauri:dev            # dev, ISOLATED data dir (`pnpm tauri dev` shares prod data)
pnpm tauri:dev:ext        # same, plus symlink local extensions/ into the dev profile
cd src-tauri && cargo check && cargo clippy && cargo test
```

CI runs exactly those. It does **not** run `pnpm format:check`, which already
fails on files nobody touched, so format only your own paths
(`pnpm exec prettier --write <files>`) and never repo-wide `pnpm format`.

## Where things live

- `src-tauri/src/lib.rs`: every Tauri command (`invoke_handler`), app boot and
  CLI dispatch.
- `src-tauri/src/modules/`: one folder or file per OS resource. `pty/` runs
  terminals and `pty_daemon/` is the sidecar that keeps them alive across a
  window close; also `fs/`, `shell/`, `git/`, `ssh/`, `extensions/`, `browser/`,
  and `mcp_bridge.rs` (the local socket outside AI CLIs connect through).
- `src/app/App.tsx`: cross-module wiring only.
- `src/modules/<area>/`: every feature. `ai/` is the agent, `terminal/` the
  xterm side, `tabs/` + `panes/` + `workspaces/` the layout model, `extensions/`
  the extension host, `automation/bridge.ts` everything an outside driver may
  call in-realm.
- `src/settings/`: the Settings window, a SEPARATE webview. Its state layer is
  `src/modules/settings/`, shared with the main window through the store.
- `scripts/`: the verify suite (one folder per subsystem), `mcp/` (the stdio MCP
  server and its shared tool table) and `release/`.
- `extensions/`: gitignored working copies of the extension repos. Only
  `README.md`, `tedi.d.ts` and `manifest.schema.json` are committed here.

## Architecture rules

Load-bearing. Breaking one is a bug, not a style question.

- **Two processes.** The webview reaches the OS only through
  `invoke("cmd", args)`; streaming comes back over a Tauri `Channel`. Every
  command is registered in `src-tauri/src/lib.rs` `invoke_handler` - that list is
  the whole backend API surface, so read it there rather than trusting any doc.
- **Import through the `@/*` alias only**, never a relative path that leaves your
  own module. Enforced by `pnpm lint:imports`. A module's `index.ts` is a
  convenience re-export, not a required door.
- **Tabs never unmount.** Inactive tabs hide with `invisible pointer-events-none`
  so PTYs and dev servers keep streaming. `visibility:hidden` KEEPS the layout
  box, so a size check reports "visible" for a pane nobody is showing and an
  `IntersectionObserver` never fires.
- **Secrets live only in the OS keychain** (`secrets_*`, service `tedi`). Never on
  disk, in the settings store, or in `localStorage`.
- **Extensions are not sandboxed.** Their JS runs in the main webview with full
  privileges, and a raw `@tauri-apps/api` import skips every `ctx` permission
  gate. The trust boundary is the install-time permission review, so a gate is a
  guard rail, never a security boundary.
- **`App.tsx` coordinates, it does not implement.** Feature logic belongs in
  `src/modules/<area>/`.
- **Tauri commands must be async.** A sync one runs on the WebView2 UI thread and
  blocking there freezes the app. A test pins the allowed-sync list.
- **The budget is RAM and idle CPU, not bundle size.** The old ~10 MB download cap
  is retired: ship what the feature needs. Resident cost is what still matters, so
  no per-tick process spawn, watch instead of poll, and gate pollers on visibility.

## Conventions that differ from the default

- **Send `\r` (CR) for Enter to a terminal, never `\n`.** PowerShell needs CR.
- **Paths**: split with `.split(/[\\/]/)`. Canonical frontend form is
  forward-slash; convert `homeDir()` backslashes at the boundary.
- **Cross-platform**: resolve HOME and cache via the `dirs` crate, never raw env
  vars. `#[cfg(not(windows))]` code never compiles on Windows, so CI is the first
  thing that sees it.
- **shadcn/ui and AI Elements are OWNED, not generated.** Add a NEW component with
  the CLI; re-running it over an existing one silently reverts TEDI's tokens.
- **Prose and docs: no em-dashes.** Use commas, colons or parentheses.
- Commit messages carry **no AI attribution**: no `Co-Authored-By` line for an
  assistant.

## Common changes

- **New Tauri command**: an `async` `#[tauri::command]` in its module, registered
  in `lib.rs` `invoke_handler`. A sync one fails the `ui_thread_guard` test. An
  extension can only call it with an `invoke:<name>` permission.
- **New AI tool**: a builder in `src/modules/ai/tools/` with an honest
  `needsApproval`, added in `buildTools` (`tools.ts`); its picker group lives in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IlhamriSKY/TEDI](https://github.com/IlhamriSKY/TEDI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
