---
trigger: always_on
description: > **Community:** https://discord.gg/Y4rDyTScPe — where we discuss OpenFight and everything around it (not only OpenFight).
---

# Repository Guidelines

> **Community:** https://discord.gg/Y4rDyTScPe — where we discuss OpenFight and everything around it (not only OpenFight).

## Project Overview

OpenFight — open-source arcade netplay platform, clean-room alternative to proprietary Fightcade. Monorepo at `D:/OpenFight` (Apache-2.0) with Tauri + React + TypeScript client and Rust + Axum + PostgreSQL server. `D:/Fightcade` (v2.1.45) is **read-only reference** — never copied into this repo (see `docs/ARCHITECTURE.md §2` and `docs/reference-fightcade-install.md`). Goal: lobby → challenge → versioned signaling → P2P (or WS relay) → safe emulator launch.

## Architecture & Data Flow

- **System:** `apps/client` (Tauri) ↔ `apps/server` (Axum) over HTTPS/WS ` /ws` → P2P/relay between peers. Server never sees game inputs except when relaying as fallback.
- **Client:** Tauri 1.x hosts React routes `Games | Lobbies | Friends | Servers | Settings`. Rust core owns `fs` (ROM scan under `emulator/<core>/ROMs/`), `process` (spawn via `tauri::api::process::Command` with arg escaping, no shell), `diag` (Network Test). State: TanStack Query + Zustand, WS client with typed `packages/protocol` discriminated unions and reconnect backoff.
- **Server:** Single Axum monolith + `postgres:16-alpine` (no Redis in MVP). Routes `POST /api/v1/auth/*`, `GET /api/v1/games`, `GET /api/v1/lobbies/:game`, `POST /api/v1/rooms` + `.../:id/{accept,decline,cancel}`, WS `/ws` with versioned envelope `{type,version,request_id,timestamp,payload}` (`presence.update`, `chat.message`, `challenge.*`, `session.offer/answer/candidate`, `room.*`). In-memory presence Hub, Postgres for durable state.
- **Networking:** Signaling relayed verbatim through server; client-driven UDP hole-punch → STUN hint (`GET /servers` returns `stun:host:port` when configured) → WS relay fallback (in-process `relay` module, future `services/relay` binary). Room states `WAITING → CHALLENGING → CONNECTING → PLAYING → FINISHED|CANCELLED`. Latency `rtt_ms/loss/jitter` via `presence.update`.
- **Reference only:** `docs/reference-fightcade-install.md` describes the opaque PyInstaller launcher (`emulator/fcade.exe`/`frm.exe` → `fightcade/launcher.py`) and `fbneo-training-mode` Lua surface — not used at runtime.

## Key Directories

- `apps/client/` — Tauri app (replaces `fc2-electron`). `src/routes/{Games,Lobbies,Friends,Servers,Settings}.tsx`, `src/components/*`, `src/lib/{api,ws,store}.ts`, `src-tauri/src/{main.rs,commands/{fs,process,diag}.rs,adapters/fbneo.rs}`, `tauri.conf.json` (least-privilege `fs`/`process` allowlist).
- `apps/server/` — Axum monolith (replaces `fightcade.com/replay`). `src/{main.rs,routes/*,ws.rs,state.rs,auth.rs,models/*}`, `migrations/001_users.sql`, `Dockerfile`.
- `packages/protocol/` — shared wire types (`Envelope`, `RoomState`, `PresenceState`) — single source of truth, `serde` + `ts-rs` generation.
- `packages/emulator-sdk/` — `pub trait EmulatorAdapter { detect/validate/get_version/launch/stop/configure }` + `LaunchCtx`, `ChildHandle`. No shell, path canonicalization + prefix check.
- `packages/game-definitions/` — declarative `games/*.toml` (`schema_version=1`, `id`, `name`, `emulator="fbneo"`, `[launch] args=["{rom}"]`, `[validation] required_files=["neogeo.zip"]`) + `src/loader.rs` + legacy `emulator/*.json` → TOML importer (build-time only).
- `adapters/fbneo/` — only required adapter in MVP (`fcadefbneo.exe` detection, `fcadefbneo.default.ini` version check, safe arg building). Future `flycast`/`snes9x` behind feature flags.
- `services/relay/` — placeholder crate `openfight-relay` (future STUN/TURN); not required for MVP (in-process WS relay suffices).
- `research/` — **not shipped** — `observations/`, `protocol/`, `binaries/` (gitignored), `network/`, `behavior/`, `notes/` + `GUARDRAILS.md`. Keep `research/binaries/.gitkeep`.
- `docs/` — `ARCHITECTURE.md` (authoritative), `reference-fightcade-install.md` (read-only install notes).
- `tests/`, `docker/`, `.github/workflows/` — integration tests, compose overlay, CI.
- Reference (read-only, outside repo): `D:/Fightcade/emulator/fbneo/fbneo-training-mode/` — vendored Lua `games/<rom>/<rom>.lua` + `hitboxes/*.lua` + `Run()` hook; pattern only.

## Development Commands

```bash
# prerequisites: Rust 1.78+, pnpm 9+, Docker, Postgres 16
pnpm install                    # workspace install (apps/*, packages/*)
pnpm -C apps/client tauri dev   # Vite + Tauri dev (Windows)
pnpm -C apps/client build       # TS check + Vite build
pnpm format && pnpm lint        # prettier + eslint
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p openfight-server -- --migrate   # sqlx migrate run
cargo run -p game-defs-import -- D:/Fightcade/emulator/fbneo_roms.json --out packages/game-definitions/games

# infra
docker compose -f docker/docker-compose.yml up --build -d   # or docker-compose.yml at root
curl http://localhost:8080/health && curl http://localhost:8080/ready
psql $DATABASE_URL -f apps/server/migrations/001_users.sql

# diagnostics
pnpm tauri dev -- --log openfight  # client logs to %APPDATA%/OpenFight/logs/openfight.log
```

## Code Conventions & Common Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zendevve/OpenFight](https://github.com/Zendevve/OpenFight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
