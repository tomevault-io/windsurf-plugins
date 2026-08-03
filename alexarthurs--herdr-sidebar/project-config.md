---
trigger: always_on
description: **This file is a living doc — always capture findings.** Whenever you discover something
---

# herdr-sidebar monorepo

**This file is a living doc — always capture findings.** Whenever you discover something
non-obvious the hard way (a herdr behavior, a Windows quirk, a manifest gotcha, a build issue),
record it here in the relevant section before finishing the task, the way the Windows caveats
below were captured. If you're working in a feature worktree, commit the CLAUDE.md update on
your branch so it lands on main with the merge.

One herdr plugin, a VS Code-style sidebar for the terminal, as a **self-contained Rust crate**:

- `plugins/herdr-sidebar` — file explorer + source control in ONE binary (ratatui TUI).
  Unified mode shows both views in a single "Sidebar" pane with an activity-bar switcher
  (in-process, instant); the ⚙ settings can split them into separate Explorer /
  Source Control panes (`--view explorer|git` pins a pane's starting view). `--preview`
  runs the file-preview pane. Views live in `src/explorer_app.rs` / `src/scm_app.rs`
  (bin modules); shared pieces (icons, ipc, launch parsing, state, ui helpers) are lib
  modules — nothing is copy-mirrored anymore.

There is deliberately **no root cargo workspace**: `herdr plugin install <owner>/<repo>/<subdir>`
treats the subdirectory as the plugin root, and each plugin's `herdr-plugin.toml` points at
`./target/release/<bin>` — a shared workspace would hoist `target/` to the repo root and break
that path. Keep every crate buildable standalone from its own directory.

## Build / test / lint

Run from inside the plugin directory, not the repo root:

```
cd plugins/herdr-sidebar
cargo build --release
cargo test
cargo clippy -- -D warnings
```

## Plugin dev workflow

- `herdr plugin link .` (from the plugin dir) registers the local checkout with the running
  herdr; `herdr plugin list --json` shows what's registered.
- `herdr plugin action list` / `herdr plugin action invoke <plugin>.<action>` run manifest actions.
- `herdr plugin log list --plugin <id>` shows plugin logs.
- Manifest format: `herdr-plugin.toml` (`[[build]]`, `[[panes]]`, `[[actions]]`).
- herdr.dev/docs/plugins lists an `[[actions]]` `contexts` field as REQUIRED, but no
  working plugin ships it (checked herdr-file-viewer, herdr-spreader, ours, herdr-notes)
  — doc/implementation drift; leave it out.

### Reference implementations (installed locally, read these before designing)

- `%APPDATA%\herdr\plugins\github\herdr-file-viewer-c993314e2614\` — a mature git-aware file
  viewer plugin (ratatui). Its `herdr-plugin.toml` header documents hard-won **Windows
  findings** — read it before touching manifests.
- `%APPDATA%\herdr\plugins\github\herdr-spreader-f248c87aa2e2\` — minimal manifest + layout tool.
- herdr source: https://github.com/ogulcancelik/herdr — **if you run into issues integrating a
  plugin** (manifest not loading, pane spawn failures, action/IPC behavior that doesn't match the
  docs), read the open-source herdr code there to see what the host actually does, rather than
  guessing from error messages.

### Windows caveats (verified by herdr-file-viewer against herdr 0.7.1)

- herdr **cannot spawn a relative `[[panes]]` command on Windows** — it resolves the program
  against herdr's own directory and fails with ERROR_PATH_NOT_FOUND. Windows launches must go
  through an action script that spawns the binary **by absolute path** (`pane split` +
  `pane run`), locating the plugin root via `herdr plugin list --json` (strip the `\\?\` prefix).
- Action ids must be **globally unique** across platforms — use `-windows`-suffixed ids for
  the Windows variants and gate both with the item-level `platforms` key.
- herdr panes on this machine run **Windows PowerShell 5.1**: chain with `;` / `if ($?)`,
  never `&&`.
- **PS 5.1 mangles native-command arguments containing double quotes** — even inside a
  single-quoted here-string: `git commit -m @'…"quoted text"…'@` splits the message at the
  embedded `"` into multiple pathspec args (bit an agent live). Write multi-line/quoted
  commit messages to a temp file and use `git commit -F <file>` instead.
- **PS 5.1 prepends a UTF-8 BOM when piping into a native process's stdin** (`$json | my.exe`
  delivers `EF BB BF{...}`; verified live by both plugins). serde_json rejects a BOM before
  `{`, so anything parsing herdr JSON from stdin must strip a leading `\u{feff}` first (see
  `strip_bom` in both plugins' `launch.rs`).
- `cargo build --release` fails with **os error 5 (Access is denied)** while the plugin's TUI is
  running in a pane — Windows locks running exes. Close/quit the pane first, rebuild, relaunch.
- **Propagating a rebuild to every workspace**: plugin registration is global (one `plugin link`
  serves all workspaces), but stale panes keep old binaries AND a dead-but-open Explorer/Sidebar
  pane blocks the ensure hook's re-dock (it matches by label/token, not liveness). Run
  `herdr plugin action invoke herdr-sidebar.redeploy-windows` after rebuilding: it closes
  every herdr-aa pane in every workspace, kills stragglers, and re-docks the focused workspace;
  the others re-dock via the focus hook the moment they're next visited.

### herdr behavior findings (verified live against herdr 0.7.1)

Pane geometry & CLI semantics:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexarthurs/herdr-sidebar](https://github.com/alexarthurs/herdr-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
