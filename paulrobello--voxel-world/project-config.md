---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
make build          # Build release (default)
make run            # Build and run release
make run-debug      # Build and run debug with RUST_BACKTRACE=1
make test           # Run tests
make fmt            # Format code
make lint           # Run clippy linter
make checkall       # Format, lint, and test (run after making changes)
make sprite-gen     # Generate palette/hotbar sprites
make run-cap-exit   # Run, screenshot at 3s, exit at 4s (for visual debugging)
make new-flat       # Reset and create flat world (seed 314159)
make new-normal     # Reset and create normal world (seed 314159)
```

The Makefile sets `DYLD_LIBRARY_PATH` and `VK_ICD_FILENAMES` for macOS MoltenVK — run the binary via `make` or replicate those env vars when invoking `./target/release/voxel-world` directly.

### Multiplayer

```bash
make run-host       # Start as LAN host on default port
make run-client     # Join LAN host
make reset-host     # Reset host-side save/state
make reset-client   # Reset client-side save/state
```

### Benchmarking

All benchmarks use `--auto-fly` so the player auto-travels through the world and exercises chunk streaming / origin shifts. CSV profile data is written to `profiles/`.

```bash
make benchmark          # Flat terrain, 45s @ 2x speed
make benchmark-hills    # Hilly terrain, 45s
make benchmark-spiral   # Spiral flight pattern, 90s
make benchmark-normal   # Realistic normal terrain, 45s
make benchmark-stress   # Full-speed stress run
make benchmark-cap      # Short run with screenshot
make benchmark-compare ARGS="profiles/a.csv profiles/b.csv"  # Thermal-aware A/B
```

Benchmark runs sample Apple Silicon thermals and on-battery state into the CSV; `scripts/bench_stats.sh` filters noisy samples when comparing.

### Quality & Profile Targets

```bash
make run-potato | run-low | run-medium | run-high | run-ultra   # Quality presets (--quality flag)
make auto-profile-flat                                          # 45s automated feature-flag cycle (flat)
make auto-profile-normal                                        # 45s automated feature-flag cycle (normal)
make run-p1     # or run-p2                                     # Multi-instance (isolated data_p1/, data_p2/)
```

## CLI Options

```bash
make run ARGS="--seed 42"                   # Custom terrain seed (-S)
make run ARGS="--fly-mode"                  # Start in fly mode (-f)
make run ARGS="--spawn-x 100 --spawn-z 200" # Custom spawn (-x, -z)
make run ARGS="--time-of-day 0.5"           # Pause at noon (-t)
make run ARGS="--view-distance 8"           # Increase view distance (-v)
make run ARGS="--render-mode depth"         # Start in depth mode (-r)
make run ARGS="--world-gen flat"            # Flat world generation
make run ARGS="--auto-fly"                  # Auto-traverse — used by benchmarks
make run ARGS="--verbose"                   # Debug output
make run ARGS="--screenshot-delay 5"        # Screenshot after 5s (-s)
make run ARGS="--exit-delay 10"             # Exit after 10s (-e)
```

### Perf Tuning Env Vars

Chunk streaming and GPU upload budgets are overridable at runtime — useful for reproducing stalls on slower machines or isolating a regression.

| Variable | Default | Purpose |
|---|---|---|
| `ORIGIN_SHIFT_PROFILE=1` | off | Per-shift `[ShiftProfile]` / `[UploadProfile]` timing on stderr |
| `ORIGIN_SHIFT_NEAR_RADIUS` | `view_distance` | Sync-upload radius on origin shift. Smaller = less stall but visible pop-in |
| `METADATA_CHUNKS_PER_FRAME` | 128 | Chunks whose SVT metadata is rebuilt per frame |
| `METADATA_RESET_BUDGET` | 256 | Per-frame budget during a full metadata reseed after a shift |
| `REUPLOAD_PER_FRAME` | 256 | Chunks drained from `reupload_queue` per frame after a shift |
| `UPLOADS_PER_FRAME` | 256 | Dirty chunks uploaded to GPU per frame |

## Development Workflow

### WARNING: PRIORITY ONE: Commit After Every Batch of Work

**CRITICAL**: To enable rollback and prevent loss of working states:

1. After completing a logical batch of changes, run `make checkall`
2. Fix any errors or warnings found
3. **Commit immediately** with a descriptive message
4. Do NOT accumulate multiple unrelated changes before committing

```bash
make checkall                    # Must pass before committing
git add -A
git commit -m "type: description"
```

### Code Quality Check

**IMPORTANT**: After making any code changes, always run:
```bash
make checkall
```

The project is not ready until `make checkall` passes without errors.

### Logging

Production code uses the `log` crate macros (`log::debug!` / `log::info!` / `log::warn!` / `log::error!`). `println!` and `eprintln!` appear only inside `#[cfg(test)]` blocks — do not reintroduce them in production paths. Existing prefixes like `[Server]`, `[Client]`, `[World Gen]` are kept inside the format string; the severity is carried by the macro choice.

### `.unwrap()` policy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulrobello/voxel-world](https://github.com/paulrobello/voxel-world) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
