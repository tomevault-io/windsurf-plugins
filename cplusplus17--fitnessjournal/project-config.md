---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fitness Journal Coach — an automated AI fitness coaching ecosystem written in Rust (backend) and Next.js (dashboard). It fetches health/activity data from Garmin Connect, generates personalized weekly workout plans via Google Gemini AI, uploads them to the Garmin calendar, and communicates with the user through a Signal Messenger bot. A Next.js PWA dashboard provides rich visualization including muscle heatmaps, recovery charts, strength progression tracking, and AI coaching chat.

## Build & Development Commands

### Rust Backend (root directory)
```bash
cargo build                    # Debug build
cargo build --release          # Release build
cargo run -- --api             # Start REST API server (port 3001)
cargo run -- --signal --daemon # Start Signal bot + background daemon
cargo run -- --login           # Interactive Garmin OAuth login
cargo run -- --delete-workouts # Delete all FJ-AI: prefixed workouts from Garmin
cargo run -- --test-upload <file.json>  # Test uploading a workout file
cargo run -- --test-fetch <workout_id>  # Fetch and print a specific workout
cargo run -- --test-fetch-url <url>     # Fetch an arbitrary Garmin URL
cargo run -- --test-refresh    # Test OAuth2 token refresh
cargo fmt --all -- --check     # Format check
cargo clippy --all-targets --all-features -- -D warnings  # Lint
cargo test --all-targets       # Run tests
```

### Next.js Dashboard (`dashboard/`)
```bash
cd dashboard
npm install        # Install dependencies
npm run dev        # Dev server (port 3000)
npm run build      # Production build
npm run lint       # ESLint
```

### Full Preflight (both Rust + dashboard)
```bash
./scripts/publish-preflight.sh
```

### Docker
```bash
docker-compose up -d --build   # Build and start all 4 services
```

## Architecture

### Rust Backend (`src/`)
Single binary with multiple runtime modes selected via CLI flags (`clap`):
- `--api` — Axum REST API server for the dashboard
- `--signal` — Signal bot WebSocket listener
- `--daemon` — Background loop (5-min cycle): fetches Garmin data, syncs to SQLite, triggers AI analysis/generation
- `--login` — Interactive Garmin OAuth flow with MFA support
- `--delete-workouts` — Bulk delete AI-managed workouts from Garmin
- `--test-upload`, `--test-fetch`, `--test-fetch-url`, `--test-refresh` — Debug utilities

Key modules:
- **`config.rs`** — `AppConfig` loaded via `figment` (merges `Fitness.toml` → `Fitness.json` → env vars). Supports profiles (`[default]`, `[dry_run]`). Contains all timing config for notifiers, rate limits, and API bind address.
- **`garmin_api.rs`** — Native Rust Garmin Connect API client (OAuth1/OAuth2). Endpoints: activities, exercise sets, training plans, user profile, max metrics, calendar, workouts (CRUD), sleep data, body battery, training readiness, HRV status, RHR trend. Handles automatic OAuth2 token refresh.
- **`garmin_client.rs`** — High-level client wrapping `GarminApi`. Fetches and assembles `GarminResponse` (activities with set details, plans, profile, metrics, scheduled workouts, recovery). Caches responses in SQLite (5-min TTL). Manages AI workout lifecycle: `cleanup_ai_workouts()`, `create_and_schedule_workout()`, `validate_and_fix_strength_workouts()` (checks scheduled workouts match generated specs), `workout_steps_match()`.
- **`garmin_login.rs`** — Garmin SSO login flow: credentials → CSRF ticket → OAuth1 token → OAuth2 exchange. Full MFA support with `login_step_2_mfa()`.
- **`ai_client.rs`** — Gemini API client. Two modes: single-shot `generate_workout()` and multi-turn `chat_with_history()` with system instruction and context injection. Configurable model via `GEMINI_MODEL` env var (default: `gemini-3-flash-preview`). Logs token usage from response metadata. Includes `extract_json_block()` for parsing workout JSON from markdown responses.
- **`coaching.rs`** — `Coach` builds the comprehensive text "brief" (prompt) from Garmin data, profile goals/constraints/equipment, progression history, weekly deltas, adherence tracking, previous plan response (coaching memory), and recent activity analyses. Also contains `generate_smart_plan()` for training plan logic.
- **`bot.rs`** — Signal bot controller:
  - **WebSocket listener** to `signal-cli-rest-api` with note-to-self/syncMessage support and message deduplication (rolling 100-message buffer).
  - **Commands**: `/status` (body battery, sleep, today's plan), `/generate` (trigger full coach pipeline), `/macros <kcal> <protein>` (log nutrition), `/readiness` (AI race readiness assessment).
  - **Free-text conversation**: Gemini-powered chat with persistent history in SQLite. Context-enriched with: body battery, sleep, today's workouts, 7-day activities, 7-day coach feedback, upcoming races/events with countdown, profile goals/constraints/equipment, and top 15 all-time strength PRs. Can auto-schedule workouts from conversational responses.
  - **Scheduled notifiers** (all broadcast to subscribers):
    - Morning Briefing — daily at `morning_message_time`, lists today's workouts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CPlusPlus17/FitnessJournal](https://github.com/CPlusPlus17/FitnessJournal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
