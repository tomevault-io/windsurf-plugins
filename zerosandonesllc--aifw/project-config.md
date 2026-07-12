---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AiFw is an open-source (MIT) AI-powered firewall for FreeBSD, built in Rust on top of `pf`. All features are free. AI/ML threat detection is optional — it works as a traditional firewall without it.

## Build & Test Commands

```bash
# Rust
cargo check                    # type check (must pass with zero warnings before commit)
cargo build --release          # release binaries
cargo test                     # run all tests (~273 tests)
cargo test test_login          # run a single test by name
cargo test --package aifw-core # run tests for one crate

# Web UI
cd aifw-ui
npm ci                         # install dependencies
npm run build                  # static export to aifw-ui/out/
npm run dev                    # dev server on :3000
npm run lint                   # ESLint
```

Release binaries: `target/release/{aifw,aifw-daemon,aifw-api,aifw-tui,aifw-setup}`

## Version Management

All components share one version. Bump BOTH before every commit:

| File | Field |
|------|-------|
| `Cargo.toml` (root) | `[workspace.package] version` |
| `aifw-ui/package.json` | `"version"` |

Increment: major=breaking, minor=features, patch=fixes.

## Architecture

### Crate Dependency Flow

```
aifw-common (shared types: rules, NAT, VPN, TLS, geo-IP, HA, IDS, metrics)
    ↑
aifw-pf (PfBackend trait + mock/ioctl implementations)
    ↑
aifw-core (engines: rules, NAT, VPN, geo-IP, HA, shaping, audit, DB)
    ↑
aifw-conntrack / aifw-plugins / aifw-ai / aifw-ids / aifw-metrics
    ↑
aifw-api (Axum REST API) / aifw-daemon / aifw-cli / aifw-tui / aifw-setup
```

### PfBackend: Cross-Platform Development

The `PfBackend` trait (`aifw-pf/src/backend.rs`) abstracts pf operations. Backend selection is **compile-time** via `#[cfg(target_os)]`, not feature flags:

- **Linux/WSL** (`mock.rs`): `PfMock` — in-memory rule storage for development/testing
- **FreeBSD** (`ioctl.rs`): `PfIoctl` — real pfctl commands via sudo

`aifw_pf::create_backend()` returns the correct implementation automatically. All development and testing works on Linux/WSL with the mock backend.

### Engine Pattern

All core engines (`aifw-core/src/`) follow the same structure:

```rust
pub struct XEngine {
    pool: SqlitePool,           // database handle
    pf: Arc<dyn PfBackend>,     // shared pf backend
    anchor: String,             // pf anchor name (e.g., "aifw", "aifw-nat")
}
```

Engines: `RuleEngine` (engine.rs), `NatEngine` (nat.rs), `AliasEngine` (alias.rs), `GeoIpEngine` (geoip.rs), `VpnEngine` (vpn.rs), `ShapingEngine` (shaping.rs), `HaEngine` / `ClusterEngine` (ha.rs), plus the multiwan family (`InstanceEngine`, `GatewayEngine`, `GroupEngine`, `PolicyEngine`, `LeakEngine`, `PreflightEngine`, `SlaEngine`) under `multiwan/`.

Each engine has its own `migrate()` method that creates its SQLite tables. Migrations are **inline SQL** in Rust code (no separate migration files).

### Database Layer

Central type: `Database` struct in `aifw-core/src/db.rs` wrapping `SqlitePool`.

- `Database::new(path)` — file-based SQLite
- `Database::new_in_memory()` — for tests
- Production path: `/var/db/aifw/aifw.db`

### API Architecture

**Router** (`aifw-api/src/main.rs` → `build_router()`): Three-tier routing:

1. **Public routes** — `/auth/login`, `/auth/register`, OAuth callbacks (no auth)
2. **Admin routes** — user management, config import/export, updates (`require_admin` middleware)
3. **Protected routes** — everything else (`auth_middleware` only)

**Auth middleware** (`aifw-api/src/auth/mod.rs`) extracts identity from:
- `Authorization: Bearer <JWT>` header
- `Authorization: ApiKey <key>` header
- `?ticket=<id>` query param (WebSocket/SSE) — single-use, 30-second
  ticket issued by `POST /auth/ws-ticket` (see `auth::ws_ticket`).

**AppState** holds all engines as `Arc<T>`, shared `Arc<dyn PfBackend>`, and `SqlitePool`. Passed to handlers via Axum's `State` extractor.

### Test Patterns

Tests use in-memory SQLite and mock pf:
```rust
let db = Database::new_in_memory().await.unwrap();
let pf: Arc<dyn PfBackend> = Arc::new(aifw_pf::PfMock::new());
let engine = RuleEngine::new(db, pf);
```

API integration tests use `axum_test::TestServer` with `create_app_state_in_memory()`.

### pf Anchors

AiFw rules live in isolated pf anchors (`aifw`, `aifw-nat`, `aifw-vpn`, `aifw-geoip`, etc.), never touching system pf config.

## Web UI

Next.js 15 with `output: "export"` (static HTML, no Node.js on appliance). Tailwind CSS 4, TypeScript. Served by the API via `--ui-dir` flag on port 8080.

## External Components

`freebsd/manifest.json` is the single source of truth listing all components. AiFw depends on companion services built from separate repos:
- **TrafficCop** — reverse proxy
- **rDHCP** — DHCP server
- **rDNS** — DNS resolver
- **rTIME** — NTP/PTP time sync

The deploy script and CI pipeline build these from sibling directories or clone from GitHub.

## FreeBSD Deployment

### ISO Build (CI)
Push tag `v*` → `.github/workflows/build-iso.yml` builds UI on Linux, compiles Rust in FreeBSD VM, produces ISO + USB IMG.

### Local Build
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZerosAndOnesLLC/AiFw](https://github.com/ZerosAndOnesLLC/AiFw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
