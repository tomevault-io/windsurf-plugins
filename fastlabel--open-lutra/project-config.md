---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A ROS2-based robot data recording system. Records ROS2 topics from ROS2-compatible robots and outputs them in MCAP format. Targets robot teaching and teleoperation workflows.

## Development Commands

```bash
make setup             # Initial setup
make up                # Start the dev environment (with simulator; Frontend: :5173, Backend: :8000)
make dev-up            # Start in dev mode (VITE_DEV_MODE=true: shows developer UI such as command copy and StatusBar)
make down              # Stop
make logs              # Show logs
make ps                # Show container status
make lint              # Lint (everything: backend + frontend)
make lint-backend      # Lint (ruff + mypy)
make lint-frontend     # Lint (tsc + biome)
make test              # Test (everything: backend + frontend)
make test-backend      # Test (pytest, inside Docker)
make test-frontend     # Test (vitest)
make test-cov          # Test + coverage (everything)
make test-cov-backend  # Test + coverage (pytest, inside Docker)
make test-cov-frontend # Test + coverage (vitest)
make format            # Format (everything: backend + frontend)
make format-backend    # Format (ruff)
make format-frontend   # Format (biome)
make generate          # Regenerate API types (requires: backend running)
make build             # Build Docker images
make prod-up           # Production start (host network)
```

## Architecture

→ Details: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

A **hybrid architecture** is used:

| Function | Technology | Reason |
|------|------|------|
| Recording | `subprocess` (`ros2 bag record`) | Memory isolation; safe even for long recordings |
| Monitoring | `rclpy` (lightweight) | For real-time alerts; keeps only the latest N messages |
| Quality analysis | `mcap` Python library | Accurate metrics computed via post-hoc analysis |

## Key Technical Decisions

- **Recording runs via subprocess**: `ros2 bag record -s mcap` runs in a separate process. No buffering on the Python side (for memory safety).
- **Output format is MCAP**: Explicitly specified via the `-s mcap` option (not the default sqlite3).
- **Robot configuration is YAML**: Topics, expected Hz, ROS_DOMAIN_ID, etc. are managed via YAML files in the `config/` directory. Switched via the `ROBOT_CONFIG` environment variable.
- **Docker required**: Both development and production run via Docker Compose.
- **Timestamps prefer header.stamp**: MCAP quality analysis, MP4 generation, and live quality (when `stamp_quality: true`) prefer `header.stamp`. This eliminates DDS delivery jitter for accurate quality evaluation. For message types without `header`, falls back to `log_time` (`backend/app/shared/stamp.py`).
- **Loss detection is IQR-based**: Instead of a fixed multiplier, uses a statistical threshold (`Q3 + 1.5×IQR`) to detect per-frame losses. Recorded as `LossEvent` (severity=minor/major) and used for per-topic status determination.
- **Image/Joint detection is automatic**: Determined by structure (presence of `format` + `data` fields), not by hard-coded message type names. Also supports vendor-specific or user-defined message types that nest a `JointState` inside a `joint_state` field. See [examples/custom_ros2_messages/](examples/custom_ros2_messages/) for how to plug in custom message packages.
- **Video preview is MCAP → MP4 conversion**: When the Preview on the recording detail page is opened, per-camera MP4s are generated from the MCAP and persisted in the recording directory. FPS is fixed at 30 (`backend/app/features/media/video_generator.py:VIDEO_FPS`). Frames are piped to ffmpeg one at a time to keep memory usage constant.
- **Feature boundaries follow the "recording lifecycle"**: `recordings` (directory operations) / `analysis` (quality and timeline; persistent findings) / `media` (MP4 / Joint data generation for preview) / `validation` (per-recording rule checks) are managed as independent features.
- **MCAP I/O is consolidated in `backend/app/infra/mcap/`**: Centralizes `make_reader` + `DecoderFactory` initialization, header.stamp-preferred timestamp normalization, and image/Joint structure detection. All consumers in analysis / media read MCAP through this layer.
- **Validation takes a ValidationContext as input**: After a recording stops, quality → validation runs automatically as a chain in JobQueue, and results are saved to `validation_result.json`. `ValidationContext` is a frozen dataclass bundling `QualityReport` / `recording_dir` / `mcap_path` / `recording_meta`; it also exposes the MCAP path so validators can read raw frames with `MCAPReader` when needed (if you only need aggregated values, `ctx.report` is enough). Builtins live in `backend/app/features/validation/builtins/` with their params controlled by `active_set.py`; user-defined validators go in `backend/app/features/validation/custom/` registered via `@register_validator` and applied on restart. See [docs/domain/custom_validators.md](docs/domain/custom_validators.md) for how to add a custom validator.

## Frontend Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fastlabel/open-lutra](https://github.com/fastlabel/open-lutra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
