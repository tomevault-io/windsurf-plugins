---
trigger: always_on
description: Instructions for Claude Code (or any AI coding agent) working in this repo.
---

# CLAUDE.md

Instructions for Claude Code (or any AI coding agent) working in this repo.

## What this is

Terminal-native, multi-agent pixel-art visualizer for AI coding agents. Each running CC (Claude Code) session shows up as an animated half-block sprite in an ASCII office. Built in Rust as a Cargo workspace of three crates.

User-facing overview: [`README.md`](README.md).
v1 spec: [`docs/superpowers/specs/2026-05-20-pixtuoid-design.md`](docs/superpowers/specs/2026-05-20-pixtuoid-design.md).
v1 plan (28 TDD-shaped tasks): [`docs/superpowers/plans/2026-05-20-pixtuoid-v1.md`](docs/superpowers/plans/2026-05-20-pixtuoid-v1.md).

## Layout

```
crates/
├── pixtuoid-core/      headless lib — no terminal deps (ratatui/crossterm forbidden here)
│   ├── source/             Source trait, hook+jsonl decoders, listeners, SourceManager
│   ├── state/              SceneState + Reducer (with Transport-tagged dedup + Active→Idle debounce)
│   ├── sprite/             .sprite parser, pack.toml loader, half-block blitter, animator
│   ├── render/             Renderer trait + TestRenderer (feature = "test-renderer")
│   ├── layout/             zone-based office geometry (terminal-agnostic):
│   │                       mod.rs (SceneLayout struct, Bounds, Point, constants, accessors),
│   │                       compute.rs (compute_with_seed + 4 private helpers),
│   │                       decor.rs (WaypointKind, WallDecor, PlantKind, PodDecor),
│   │                       mask.rs (build_walkable_mask — obstacle stamping for A*)
│   ├── pose.rs             pure state→pose derivation + wander state machine (no terminal deps)
│   ├── walkable.rs         WalkableMask (static bool grid) + OccupancyOverlay (dynamic per-frame)
│   └── tests/              one integration test per concern
├── pixtuoid/           binary — ratatui + crossterm + tokio + clap
│   ├── cli.rs              clap subcommands (run / install-hooks / uninstall-hooks / validate-pack / init-pack)
│   ├── config.rs           AppConfig persistence (~/.config/pixtuoid/config.toml), XDG-aware
│   ├── runtime.rs          tokio task wiring (source ── (Transport, AgentEvent) ──► reducer ──► renderer)
│   ├── install/            settings.json merge, atomic write, advisory lock, stow-symlink safe
│   └── tui/                ratatui App + TuiRenderer (Renderer trait impl)
│       ├── renderer.rs     draw_scene orchestrator (DrawCtx struct), half-block flush, terminal lifecycle
│       ├── widgets/        ratatui widget paint fns, split into sub-modules:
│       │                   mod.rs (TickerQueue, shared helpers), hud.rs (footer, wall display,
│       │                   elevator indicator, theme picker), tooltip.rs (hover, cat, coffee,
│       │                   furniture, labels, chitchat bubbles)
│       ├── hit_test.rs     mouse hit-test: agent hover, coffee machine click, furniture tooltips
│       ├── tui_renderer.rs Renderer trait impl — owns cross-frame state (RgbBuffer, FrameCache, Router, PoseHistory, TickerQueue, Theme, cached Layout)
│       ├── theme/          color theme system — one file per theme, Theme struct in mod.rs
│       │                   mod.rs (struct defs + ALL_THEMES registry), normal.rs, cyberpunk.rs,
│       │                   dracula.rs, tokyo_night.rs, catppuccin.rs, gruvbox.rs
│       ├── pose.rs         routed pose layer (PoseHistory, derive_with_routing, snap-back) — re-exports core::pose
│       ├── pathfind.rs     Router trait + AStarRouter with selective cache invalidation
│       └── pixel_painter/  pure-pixel pass — split into focused child modules:
│                           mod.rs (PixelCtx struct, orchestrator), background/ (weather, sunset, skyline),
│                           drawable.rs (y-sort), effects.rs (glow/z's/dots/steam/dust/bubble),
│                           palette.rs (tool_glow_tint), anchors.rs (breath, walk position, character_anchor),
│                           furniture.rs (coffee table, area rug, side table, pantry table/chair)
└── pixtuoid-hook/      tiny shim CC invokes — stdin JSON → Unix socket, 200ms write timeout
│   └── sprites/
│       ├── default/        coworking-lounge pack (embedded via include_str!)
│       ├── robot/          proof-of-concept robot character pack (loadable via --pack-dir)
│       └── skeleton/       template pack for custom sprite creation (extracted via init-pack)
scripts/                    preflight.sh (CI mirror), crop-snapshot.py (visual verification)
```

## Build & test

```
cargo build --workspace                                              # debug build
cargo build --release --workspace                                    # release build
cargo test --workspace --features pixtuoid-core/test-renderer    # all tests (200+)
cargo run --release --example snapshot -- /tmp/snap.png              # render TUI to PNG
./target/release/pixtuoid run --headless --projects-root ~/.claude/projects   # live test against real CC
```

The `test-renderer` feature is needed for the `e2e.rs` integration test. The dev workspace test alias is just `cargo test`.

### Visual verification (Python venv)

```
python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.txt
cargo build --release --example snapshot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanWng97/pixtuoid](https://github.com/IvanWng97/pixtuoid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
