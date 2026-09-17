---
trigger: always_on
description: Seance is a **human + agent co-working** app: multi-pane live terminals on
---

# seance — notes for coding agents (working on this repo)

Seance is a **human + agent co-working** app: multi-pane live terminals on
Linux (GPUI), shared scratchpads, file panes, and a Unix-socket control plane
so anyone in the circle can engage everyone else in the open. Product intent
is in `README.md` and `seance ctl skill` — not "Claude wrapper."

## Hard rules (read before anything else)

1. **Never hard-kill the daemon** (`pkill -x seance` murders live sessions).
   Deploy: `cargo build --release && seance upgrade` (runtime) or
   `seance restart-gui` (UI only). Sessions survive both.
2. **`./scripts/check.sh` must pass before every commit** — fmt --check,
   deny-warnings `cargo check --all-targets`, full test suite. The repo is at
   **zero warnings** and stays there; a warning is a build failure.
3. **Never bump gpui / zed / alacritty / gpui-component revs** casually.
   Pinned pair (bump only together, see `docs/PLAYBOOK.md`):
   GPUI patched to `deps/zed` @ `1a246efd7e1b83ab568ec5e3e6c1a43a42e1abba`
   (`./scripts/bootstrap-deps.sh`), `gpui-component` @
   `b5eef62336f88bb6c1ee45bf32f73c9895d49f8d`. Grep `deps/zed` for real APIs —
   GPUI training data is stale; **never write GPUI calls from memory**.
4. **Version lockstep is one rule, stated once.** Every workspace crate
   inherits `workspace.package.version`, and the hello line carries it: the
   daemon refuses any ctl/GUI/web client whose version isn't an exact match.
   So a version bump means, in the same breath: `./scripts/build-web.sh
   release` (rebuild the committed `crates/seance-web/dist`), `cargo build
   --release && seance upgrade` (daemon), then restart the GUI and any running
   `seance web` bridge. Skip one and that surface fails with version skew —
   by design, not a bug.
5. **Stay on `master`; never push** unless explicitly asked.
6. **Checkpoint-commit before any multi-hundred-line mechanical change.**
   A big-bang split of app.rs failed once and had to be rolled back from git.
   The discipline that then succeeded: one slice → `cargo test` → commit →
   next slice. Nothing bigger than one green step at a time.

## Architecture map (post-0.9.14 modular split; 0.11 workspace carve)

```
crates/seance-core/    sans-io shared crate — MUST compile native AND wasm32:
                       protocol.rs (wire types), snapshot.rs (SCG3 codec),
                       input.rs (key encoding), control.rs, auth.rs,
                       replay.rs (SRR1 format), util.rs (slugify),
                       links.rs (the URL under a cell — OSC-8 span, else a
                       bare http(s) run stitched across hard wraps; native
                       ctrl+click and the web's tap/ctrl+click share it).
                       util.rs also holds the rail's shared rules:
                       settle_absent (pins for circles the daemon hasn't
                       confirmed) + rail_prefs_is_foreign (own-echo guard)
crates/seance-web/     the wasm browser client: lib.rs (app core, rAF loop),
                       renderer.rs (WebGL2 atlas), conn.rs / state.rs / input.rs,
                       ui.rs + menus.rs + keymap.rs + help.rs (chrome),
                       activity.rs, probe.rs, pr_board.rs (the PR sweep
                       overlay — pure model + DOM, native prboard.rs twin),
                       replay.rs (player),
                       replay_edit.rs (editor); dist/ is committed
src/webbridge.rs       `seance web`: ws↔unix pump, token auth, static files,
                       /ws + /healthz + /replay/{list,manifest,pane,publish}
src/replayexport.rs    `seance replay` CLI + bundle exporter + publisher seam
src/runtime/recorder.rs  daemon-side replay ring recorder (48h DVR)
src/main.rs            entry: version/ctl/daemon/web/replay dispatch, SIGPIPE, window setup
src/app/               the GPUI app, split by surface:
  mod.rs      (~2.3k)  SeanceApp struct, boot, GuiEvent loop, key capture,
                       focus/rename/pane lifecycle, render() entry
  actions.rs           all Act* gpui actions + SEANCE_ARM_PROMPT
  layout.rs            layout.json load/save (pure parse/serialize split)
  util.rs              pure helpers (tips, status colors, drag types)
  chrome.rs            render_pane, help overlay, asks/activity/stage strips
  pads.rs              scratchpad drawer + phone spine
  overview.rs          ctrl+shift+space live map
  sidebar.rs           left rail: pinned rows above a rule, everything else below,
                       context menus, host list
  tiles.rs             tile grid + sashes + zoom
  palette.rs           command palette
  quicklaunch.rs       launch strip: quicklaunch chips + create/edit modal
                       (daemon-side json); also hosts the menu chips
  menus.rs             host-provided MENUS (`menus[]` of host.json): a chip
                       that runs `list_cmd` on click, drops a picker, runs
                       `select_cmd` on choice. On-demand twin of the polled
                       host widgets — see docs/HOST.md
  workspaces.rs        workspace state ops + WorkspaceAttention + band
                       partition + NavHistory (mouse back/forward visit path,
                       kept by watching the selection each render)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zackham/seance](https://github.com/zackham/seance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
