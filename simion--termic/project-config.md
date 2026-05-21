---
trigger: always_on
description: One window, many parallel agents (claude / gemini / codex) each in its own git-worktree workspace with an embedded terminal. **Performance trumps polish** — a 1-frame terminal flicker, a >100ms editor open, or an unnecessary sidebar re-render are all real regressions.
---

# termic — context for Claude Code

One window, many parallel agents (claude / gemini / codex) each in its own git-worktree workspace with an embedded terminal. **Performance trumps polish** — a 1-frame terminal flicker, a >100ms editor open, or an unnecessary sidebar re-render are all real regressions.

## Stack

React 19 + Vite 8 + TypeScript on top of Tauri 2 (Rust + WKWebView). Tailwind v4 (`@theme` CSS vars), Radix headless primitives, Zustand 5 for state (`@/store/app`, `@/store/ui`, `@/store/prefs`, `@/store/scriptRuns`). CodeMirror 6 for the editor (~150KB; **do not** swap to Monaco — verified slower in WKWebView). xterm.js + WebGL addon for terminals (DOM ribbons; canvas drops frames). portable-pty (wezterm) on the Rust side. Inter Variable for UI, JetBrains Mono Variable for code/terminal (bundled via `@fontsource-variable/*`). lucide-react + inline brand SVGs.

**No StrictMode.** Disabled in `src/main.tsx` — double-invoke races the async PTY spawn (first spawn killed before its data listener wires). Don't re-enable without auditing every async effect's cancellation.

## Layout

```
src/
├── main.tsx              (createRoot, NO StrictMode, global error → log_line)
├── App.tsx               (UnifiedBar + grid: Sidebar / MainArea / RightPanel; Settings is a z-40 overlay, NOT a replacement — preserves PTYs)
├── index.css             (@theme tokens, html.light overrides, forced grayscale smoothing)
├── lib/                  (types, ipc wrappers, review prompt, utils.cn)
├── icons/                (CliIcon + TermicLogo)
├── store/
│   ├── app.ts            (projects/workspaces/tabs + mountedWorkspaces Set + footerTerm map + bottomTabs + per-ws split/widths)
│   ├── ui.ts             (dialog visibility + busyMessage overlay)
│   ├── prefs.ts          (theme, fonts, terminal font weight, yolo, desktopNotifications)
│   └── scriptRuns.ts     (per-(ws,kind) Run/Setup live status + lines)
├── hooks/                (useShortcuts: ⌘1..9, ⌘[/], ⌘W, ⌘L, ⌘T, ⇧⌘[/] workspace nav; useAttentionNotifier)
└── components/
    ├── UnifiedBar.tsx, ui/, sidebar/, views/
    ├── workspace/{MainArea,WorkspaceView,TabBar,TerminalPane,EditorPane,DiffPane,AuxTerminal,RightPanel,FileTree}.tsx
    ├── settings/         (Appearance, General, Agents, Shortcuts, Repository)
    └── dialogs/{Dialogs,NewProject,NewWorkspace,Welcome,Review}.tsx
src-tauri/
├── tauri.conf.json       (Overlay titleBarStyle, hiddenTitle, trafficLightPosition {x:16,y:16}, visible:false → positioned then shown)
├── capabilities/default.json   (REQUIRES core:window:allow-start-dragging + allow-toggle-maximize + allow-minimize)
└── src/lib.rs            ← ALL the Rust (PTY mgr, project/workspace IO, settings, scripts, discovery, git)
```

## Run / build

```sh
npm install
npm run tauri:dev         # vite (port 1420) + cargo run; auto-rebuilds Rust on save BUT keeps old process running — quit + relaunch after Rust changes
npm run tauri:build       # release .app/.dmg in src-tauri/target/release/bundle/
npm run build             # tsc -b && vite build (type-check + bundle)
```

⌘+R after frontend changes when HMR can't push (useEffect/useState shape changes, React.lazy swaps, xterm/CodeMirror init edits). **Quit + relaunch** after `tauri.conf.json` / `capabilities/*.json` / any Rust signature change.

## Releasing

`make release` (→ `scripts/release.sh`) cuts the tag; CI does the rest. **Before** running it, add the new version's entry to the TOP of `changelog.json` (repo root) — schema is `{version, date, summary}` and you only write `summary` (`version` from the bump, `date` auto-stamped). `make release` gates on it (scaffolds a stub + aborts if `summary` is empty). The `summary` feeds the in-app Update card and Changelog dialog. CI copies `changelog.json` to `termic.dev` alongside `latest.json`. Full flow + schema: **`RELEASING.md`**. Dev the update UI with `VITE_MOCK_UPDATE=available|whatsnew npm run tauri dev`.

## Data model

- **Data dirs.** TWO directories — different owners:
  - `<data_local_dir>/termic/` (e.g. macOS: `~/Library/Application Support/termic/`) — app-owned: `projects.json`, `workspaces/`, `settings.json`. Path via `dirs::data_local_dir().join("termic")` in `lib.rs#data_dir()`.
  - `<data_local_dir>/com.simion.termic/` — tauri-plugin-state owned (window position/size from `tauri-plugin-window-state`). Path derives from `tauri.conf.json#identifier`.
- **Project** entries live in `<data_local_dir>/termic/projects.json` as a single JSON array — git repo + scripts + `preview_url` template + `files_to_copy` globs + `default_cli`.
- **Workspace** (`workspaces/<uuid>.json`, one per file) — git worktree branched from project's `base_branch`. Worktrees live at `~/termic/workspaces/<project>/<name>/`. `is_repo_root=true` workspaces point at the project's live checkout (no worktree, archive doesn't `rm -rf`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simion/termic](https://github.com/simion/termic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
