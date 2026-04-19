---
trigger: always_on
description: **projector-calib** is a Rust-based interactive calibration tool for measuring projector intrinsic parameters and lens distortion coefficients. The tool uses an **inverse distortion workflow** where users adjust parameters representing physical lens distortion, and the program computes the exact mathematical inverse using OpenCV's iterative Newton-Raphson solver.
---

# projector-calib - Claude Code Context

## Project Overview

**projector-calib** is a Rust-based interactive calibration tool for measuring projector intrinsic parameters and lens distortion coefficients. The tool uses an **inverse distortion workflow** where users adjust parameters representing physical lens distortion, and the program computes the exact mathematical inverse using OpenCV's iterative Newton-Raphson solver.

**Target Resolution**: 1920x1080 (hardcoded)

**License**: MIT (Copyright 2025 Lin Hsiang-Jui)

**Repository**: https://github.com/NEWSLabNTU/projector-calib

## Key Architectural Decisions

### Inverse Distortion Approach

The tool applies **inverse distortion** (undistortion) as pre-warp for projector calibration:

1. User adjusts distortion parameters (k1, k2, k3, p1, p2) to match the physical lens
2. Program computes the exact inverse using `opencv::calib3d::init_undistort_rectify_map`
3. Ideal pattern is pre-warped with inverse distortion
4. Projector lens applies forward distortion, canceling out the pre-warp
5. Result: geometrically correct projection

**Why inverse?** The Brown-Conrady distortion model is NOT self-inverse. Forward distortion approximations (using `-k1` for lens with `+k1`) fail for large distortions. OpenCV's `init_undistort_rectify_map` uses iterative Newton-Raphson to compute the exact inverse.

### Joint Calibration

The tool performs **joint calibration** of camera matrix and distortion coefficients:
- Scale (`s`) and translation (`tx`, `ty`) are user-adjustable
- Camera matrix is computed: `fx = fy = s × width`, `cx = width/2 + tx`, `cy = height/2 + ty`
- Physical measurement (`physical_size_meters`) is optional and set via config file

## Code Structure

```
src/
  main.rs           - Main loop, key handling, tracing initialization
  config.rs         - CalibrationParams struct, JSON serialization
  display.rs        - Display management, help overlay, hints
  distortion.rs     - Inverse distortion using OpenCV (ImageWarper)
  pattern.rs        - Pattern generation (chessboard, etc.)

docs/
  math.org          - Mathematical formulation
  system_design.org - Architecture and workflow
  roadmap.org       - Development phases and testing

Makefile            - Build targets (build, run, test, check, format, lint, clean)
Cargo.toml          - Dependencies and build profile
README.md           - User documentation
LICENSE             - MIT License
```

## Important Implementation Details

### Distortion Engine (src/distortion.rs)

Uses OpenCV's `init_undistort_rectify_map` for exact inverse computation:

```rust
calib3d::init_undistort_rectify_map(
    &camera_matrix,      // [fx, 0, cx; 0, fy, cy; 0, 0, 1]
    &dist_coeffs,        // [k1, k2, p1, p2, k3]
    &no_array(),         // R (identity)
    &no_array(),         // New camera matrix (same as input)
    Size::new(width, height),
    CV_32FC1,
    &mut map_x,
    &mut map_y,
)?;
```

**Critical**: User provides LENS distortion parameters. OpenCV computes the inverse for pre-warping.

### Configuration File (calibration.json)

Auto-created on first run. Contains:
- User-adjustable: `scale`, `tx`, `ty`, `k1`, `k2`, `k3`, `p1`, `p2`
- Computed: `fx`, `fy`, `cx`, `cy`
- Optional: `physical_size_meters`

External edits are detected via file modification time. User is prompted to press `L` to reload.

### Key Bindings

- **Scale/Translation**: `+/-`, arrow keys
- **Distortion (k1-k3)**: `W/S`, `E/D`, `R/F`
- **Distortion (p1-p2)**: `T/G`, `Y/H`
- **Step multiplier**: `[/]` (adjust step size)
- **Cell size**: `,/.` (pattern cell size)
- **Pattern cycling**: `TAB`, `Shift+TAB`
- **Actions**: `I` (toggle help), `L` (reload config), `0` (reset), `SPACE` (save), `ESC` (quit)

### Logging

Uses `tracing` crate for structured logging:
- `info!()` for user-facing messages
- `debug!()` for detailed debugging (step changes, pattern switches)
- `error!()` for errors

Default level: `info`. Override with `RUST_LOG=debug` or `RUST_LOG=trace`.

## Code Conventions

### Rust Edition

Uses **Rust 2024** features:
- Let-chain syntax for flattening nested if-let: `if let Ok(x) = foo() && let Some(y) = bar() { ... }`
- Context structs to reduce function parameter counts

### Linting

Must pass `make lint` without warnings:
- `cargo +nightly fmt --check` (formatting)
- `cargo clippy --profile release-with-debug -- -D warnings` (clippy)

Use `#[allow(dead_code)]` only for intentionally unused items kept for future use.

### Build Profile

Uses `release-with-debug` profile (defined in Cargo.toml):
- Release optimizations for performance
- Debug symbols for debugging
- No stripping

## Development Workflow

### Building and Running

```bash
make build    # Build with release-with-debug profile
make run      # Build and run in fullscreen
```

### Testing and Quality

```bash
make test     # Run tests
make check    # Run cargo check
make format   # Format code with rustfmt
make lint     # Check formatting and run clippy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NEWSLabNTU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
