---
trigger: always_on
description: Greenfield rewrite of two earlier prototypes — an editor and an in-terminal HTTP
---

# mnml — a NvChad-style terminal IDE (Rust + ratatui)

Greenfield rewrite of two earlier prototypes — an editor and an in-terminal HTTP
client — folded together. Earlier code is reference for porting logic, not a
dependency. The authoritative design notes live alongside this file (read them
before architectural decisions).

## Architecture spine — keep these load-bearing

- **Pluggable input layer.** `Box<dyn InputHandler>` (`src/input/`) translates key
  events into `Vec<EditOp>` (text editing — `src/edit_op.rs`, interpreted by the
  single chokepoint `src/editor.rs::Editor::apply`) or escalates to a small *closed*
  `AppCommand` / a registered command. The editor/buffer/render layers **never**
  branch on which handler is active — only the statusline (mode chip) and the
  cursor-shape code read the 4-variant `EditingMode`. (`grep -rn EditingMode src/ui`
  should hit only `statusline.rs`.) This is "vim way + standard way without
  conditionals everywhere" — the thing the user explicitly wants done right.
- **`Pane` + `Layout` + `Command` registry are the rest of the spine.** `Pane`
  (`src/pane.rs`) is the open-thing enum (Editor today; Pty/Request/Diff/Ai later —
  each additive). `Layout` (`src/layout.rs`) is the split tree (Empty|Leaf today;
  HSplit/VSplit in P3). `Command` (`src/command.rs`, a process-global `OnceLock`) is
  what the palette / which-key / keybindings / plugins all hang off. Adding a feature
  = register commands + maybe a `Pane`/`EditOp` variant — not a refactor.
- **Headless mode (`src/headless.rs`, renders via ratatui `TestBackend`) + the file-IPC
  channel (`src/ipc/`) share `src/app/` + `ui::draw` + `tui::dispatch_*` with the
  terminal loop (`src/tui.rs`)** so headless behavior matches the real UI. This is the
  substrate for the planned `.test` E2E format. IPC lives at `<workspace>/.mnml/ipc/`:
  `command` (JSONL host→mnml), `screen.txt` / `status.json` / `events.jsonl` (mnml→host).
- **No giant files.** App state is render-free and split across `src/app/mod.rs` plus
  per-subsystem siblings (`src/app/{git,lsp,ai,cdp,dap,…}.rs` — 25 files). `src/tui.rs`
  is *only* the crossterm event loop; chrome lives in `src/ui/`, subsystems get their
  own top-level dirs (`src/git/`, `src/http/`, `src/lsp/`, `src/ai/`, `src/cdp/`).
  Earlier prototypes' top-level files (one ~56k chars, one ~468k) both rotted
  — don't repeat that.
- Storage is a plain `String` + byte cursor in `Editor`; all mutation goes through
  `apply` so a rope can slide in later without touching call sites. Columns are chars
  for now (display-width / tabs / CJK is a P2 refinement).

## Cutting a release — the CHANGELOG secret-scrub trap

**Never write a credential-shaped literal in the CHANGELOG.** Not
`Authorization: Bearer <anything>`, not `xoxb-…`, not `sk-…`, not a
`token = "…"` line — even as an obviously-fake example.

cargo-dist embeds the CHANGELOG into `plan-dist-manifest.json`. If any
substring matches a stored repo secret's value, GitHub Actions replaces
it with `***` **inside the JSON**, which corrupts the manifest. The
`artifacts_matrix` then fails to parse, `build-local-artifacts` and
`build-global-artifacts` are silently **skipped**, and the Release run
reports **success** while shipping only `dist-manifest.json`. The tap /
winget / nfpm jobs then fail for lack of binaries.

This has now happened twice — v0.2.9 and v0.2.18. Both times the
workflow was green. Describe the shape in prose instead ("an auth
header written as a `{{VAR}}` reference").

**After every release, verify assets — a green run is not enough:**

```bash
gh release view vX.Y.Z --json assets --jq '.assets|length'   # want ~22, not 1
```

A tag that shipped no binaries can't be reused safely; bump the patch
version and re-cut (v0.2.9 → v0.2.10, v0.2.18 → v0.2.19).

## Build / run / test

```bash
cargo build            # debug
cargo test             # unit tests
cargo clippy --all-targets   # must be warning-free
cargo fmt              # before committing

./run.sh               # launch mnml in *your* cwd (build + run, relaunch-on-exit-75 loop)
./run.sh ~/some/proj   # launch on a specific workspace
./run.sh restart       # tell the running mnml to rebuild + relaunch (IPC {"cmd":"restart"})
./run.sh stop          # quit the running mnml
./run.sh status        # show the marker (workspace, IPC dir)
./run.sh headless [WS]  # same loop, but --headless (virtual screen + file-IPC)
./run.sh shot [OUT.png] # screenshot the *real* ghostty window (live pixels) → PNG you can Read
./run.sh clean [mode]   # reclaim target/ space — incremental (default, safe) | deps | all
./run.sh watch         # cargo-watch auto-rebuild-on-save loop (needs `cargo install cargo-watch`)
./run.sh menu          # interactive numbered picker (standalone/headless/watch/build/…)

cargo run -- [WS] [--input vim|standard] [--ascii] [--config PATH] [--headless]
cargo run -- run FILE [--env NAME]    # HTTP: send a .http/.curl/.rest file headlessly
cargo run -- chain run FILE           # HTTP: run a .chain.json
cargo run -- discover SPEC [--out DIR]  # HTTP: OpenAPI/Swagger → .curl stubs
cargo run -- test [PATH…]             # run .test E2E scripts (default tests/e2e/); also under `cargo test`
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chris-mclennan/mnml](https://github.com/chris-mclennan/mnml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
