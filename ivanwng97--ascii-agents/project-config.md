---
trigger: always_on
description: Instructions for Claude Code (or any AI coding agent) working in this repo.
---

# CLAUDE.md

Instructions for Claude Code (or any AI coding agent) working in this repo.

## What this is

Terminal-native, multi-agent pixel-art visualizer for AI coding agents. Each running CC (Claude Code) session shows up as an animated half-block sprite in an ASCII office. Built in Rust as a Cargo workspace of three crates.

User-facing overview: [`README.md`](README.md).
v1 spec: [`docs/superpowers/specs/2026-05-20-ascii-agents-design.md`](docs/superpowers/specs/2026-05-20-ascii-agents-design.md).
v1 plan (28 TDD-shaped tasks): [`docs/superpowers/plans/2026-05-20-ascii-agents-v1.md`](docs/superpowers/plans/2026-05-20-ascii-agents-v1.md).

## Layout

```
crates/
├── ascii-agents-core/      headless lib — no terminal deps (ratatui/crossterm forbidden here)
│   ├── source/             Source trait, hook+jsonl decoders, listeners, SourceManager
│   ├── state/              SceneState + Reducer (with Transport-tagged dedup + Active→Idle debounce)
│   ├── sprite/             .sprite parser, pack.toml loader, half-block blitter, animator
│   ├── render/             Renderer trait + TestRenderer (feature = "test-renderer")
│   ├── layout/             zone-based office geometry (terminal-agnostic):
│   │                       mod.rs (SceneLayout::compute, Bounds, Point, constants),
│   │                       decor.rs (WaypointKind, WallDecor, PlantKind, PodDecor),
│   │                       mask.rs (build_walkable_mask — obstacle stamping for A*)
│   ├── pose.rs             pure state→pose derivation + wander state machine (no terminal deps)
│   ├── walkable.rs         WalkableMask (static bool grid) + OccupancyOverlay (dynamic per-frame)
│   └── tests/              one integration test per concern
├── ascii-agents/           binary — ratatui + crossterm + tokio + clap
│   ├── cli.rs              clap subcommands (run / install-hooks / uninstall-hooks)
│   ├── runtime.rs          tokio task wiring (source ── (Transport, AgentEvent) ──► reducer ──► renderer)
│   ├── install/            settings.json merge, atomic write, advisory lock, stow-symlink safe
│   └── tui/                ratatui App + TuiRenderer (Renderer trait impl)
│       ├── renderer.rs     draw_scene orchestrator, half-block flush, label/tooltip widgets, footer, TickerQueue, neon wall display
│       ├── tui_renderer.rs Renderer trait impl — owns cross-frame state (RgbBuffer, FrameCache, Router, PoseHistory, TickerQueue, Theme)
│       ├── theme/          color theme system — one file per theme, Theme struct in mod.rs
│       │                   mod.rs (struct defs + ALL_THEMES registry), normal.rs, cyberpunk.rs,
│       │                   dracula.rs, tokyo_night.rs, catppuccin.rs, gruvbox.rs
│       ├── pose.rs         routed pose layer (PoseHistory, derive_with_routing, snap-back) — re-exports core::pose
│       ├── pathfind.rs     Router trait + AStarRouter with selective cache invalidation
│       └── pixel_painter/  pure-pixel pass — split into focused child modules:
│                           mod.rs (orchestrator), background.rs, drawable.rs
│                           (y-sort), effects.rs, palette.rs (tool_glow_tint), anchors.rs
└── ascii-agents-hook/      tiny shim CC invokes — stdin JSON → Unix socket, 200ms write timeout
│   └── sprites/default/    coworking-lounge pack (embedded via include_str!): seated, typing ×2,
│                           standing, walking ×2, walking_back ×2, working_couch ×2,
│                           working_floor ×2, sitting_couch, back_couch, seated_floor,
│                           sleeping variants, desk, plant ×4, cat (walk/sit/sleep),
│                           pantry, door ×3, meeting_sofa, bookshelf, whiteboard, tv_stand, etc.
scripts/                    preflight.sh (CI mirror), crop-snapshot.py (visual verification)
```

## Build & test

```
cargo build --workspace                                              # debug build
cargo build --release --workspace                                    # release build
cargo test --workspace --features ascii-agents-core/test-renderer    # all tests (147+)
cargo run --release --example snapshot -- /tmp/snap.png              # render TUI to PNG
./target/release/ascii-agents run --headless --projects-root ~/.claude/projects   # live test against real CC
```

The `test-renderer` feature is needed for the `e2e.rs` integration test. The dev workspace test alias is just `cargo test`.

### Visual verification (Python venv)

```
python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.txt
cargo build --release --example snapshot
./target/release/examples/snapshot --cols 192 --rows 64 /tmp/snap.png
.venv/bin/python3 scripts/crop-snapshot.py /tmp/snap.png --scale 3
```

See `.claude/skills/beautify-decoration/SKILL.md` for the full iteration loop, self-critique checklist, and sprite-format pitfalls.

### Pre-push preflight

`scripts/preflight.sh` mirrors `.github/workflows/ci.yml` (rustfmt + clippy with
`-D warnings` + workspace tests). Run it locally to avoid the round-trip of
"push → wait for CI → red → fix → push again."

`.githooks/pre-push` calls it automatically. Activate the hook **once per
clone**:

```
git config core.hooksPath .githooks
```

Bypass in an emergency with `git push --no-verify` or `SKIP_PREFLIGHT=1 git push`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanWng97/ascii-agents](https://github.com/IvanWng97/ascii-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
