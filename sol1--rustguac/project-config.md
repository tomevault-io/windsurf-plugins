---
trigger: always_on
description: rustguac is a lightweight Rust replacement for the Apache Guacamole Java webapp. It proxies the Guacamole protocol over WebSockets between web browsers and guacd (the C daemon from guacamole-server). Supports SSH, RDP, VNC, web browser sessions (headless Chromium on Xvnc), and VDI desktop containers (Docker).
---

# CLAUDE.md — Project state for rustguac

## What this project is

rustguac is a lightweight Rust replacement for the Apache Guacamole Java webapp. It proxies the Guacamole protocol over WebSockets between web browsers and guacd (the C daemon from guacamole-server). Supports SSH, RDP, VNC, web browser sessions (headless Chromium on Xvnc), and VDI desktop containers (Docker).

## Architecture

- **Rust binary** (`rustguac`) — axum web server, session manager, WebSocket proxy
- **guacd** — built from apache/guacamole-server source, handles SSH/VNC/RDP protocol translation
- **Xvnc + Chromium** — spawned per web-browser session, streamed via VNC through guacd
- **Docker** — VDI containers spawned per-user, connected via RDP through guacd

## Key files

- `src/main.rs` — entry point, CLI (clap), server setup
- `src/api.rs` — REST API endpoints (session CRUD, recordings, admin)
- `src/session.rs` — session state machine, SessionManager
- `src/browser.rs` — Xvnc + Chromium process lifecycle (display allocator, per-session profile dirs)
- `src/vdi/mod.rs` — VdiDriver trait, container types (ContainerSpec, ContainerInfo, ManagedContainer)
- `src/vdi/docker.rs` — Docker-based VDI driver (bollard, unix socket, start/reuse/stop)
- `src/guacd.rs` — TCP connection to guacd, Guacamole protocol handshake
- `src/protocol.rs` — Guacamole wire format parser/encoder
- `src/websocket.rs` — WebSocket <-> guacd TCP bridge, recording tee
- `src/config.rs` — TOML config loading with defaults
- `src/auth.rs` — API key auth middleware (SHA-256, IP allowlists, expiry), role system
- `src/oidc.rs` — OIDC authentication (login, callback, logout, group extraction)
- `src/vault.rs` — Vault/OpenBao KV v2 client for connections (AppRole auth, token renewal)
- `src/db.rs` — SQLite admin database (rusqlite, bundled)
- `static/client.html` — Guacamole JS client with auto-scaling display
- `static/connections.html` — Vault-backed connections UI (folder/entry management, connect)
- `static/recordings.html` — recording playback with auto-scaling
- `static/sessions.html` — session management dashboard
- `dev.sh` — development script (build guacd, run, deps)
- `install.sh` — bare-metal Debian 13 installer (systemd services)
- `Dockerfile` — multi-stage build (guacd + rustguac + runtime)

## Configuration

TOML config file (`config.local.toml` for dev, `--config` flag for production). Key settings: `listen_addr`, `guacd_addr`, `recording_path`, `static_path`, `db_path`, `xvnc_path`, `chromium_path`, `display_range_start/end`.

### Vault / Connections

Optional `[vault]` section enables the Vault-backed connections. Connection entries (SSH/RDP/Web) are stored in Vault KV v2 — credentials never touch disk or the browser.

```toml
[vault]
addr = "https://vault.example.com:8200"
mount = "secret"           # KV v2 mount (default)
base_path = "rustguac"     # base path under mount (default)
role_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
# namespace = "my-ns"      # optional, for Vault Enterprise / OpenBao namespaces
# instance_name = "prod-1" # optional, enables instance-scoped entries
```

`VAULT_SECRET_ID` env var provides the AppRole secret ID.

Vault KV v2 path structure:
- `<base_path>/shared/<folder>/<entry>` — shared across all instances
- `<base_path>/instance/<name>/<folder>/<entry>` — instance-specific
- `<folder>/.config` — folder metadata: `{"allowed_groups":["group1"], "description":"..."}`

### OIDC

Optional `[oidc]` section enables OpenID Connect authentication. Key settings: `issuer_url`, `client_id`, `client_secret`, `redirect_uri`. `OIDC_CLIENT_SECRET` env var can override the config value. `groups_claim` (default: "groups") specifies the JWT claim for group memberships. `extra_scopes` requests additional scopes.

### Roles

4-tier role hierarchy: `admin` (4) > `poweruser` (3) > `operator` (2) > `viewer` (1).
- **admin**: full access, connections folder/entry management
- **poweruser**: ad-hoc session creation + connections connect
- **operator**: connections connect only (no ad-hoc sessions)
- **viewer**: read-only

## Deployment

- **Bare metal**: `sudo ./install.sh` on Debian 13. Installs to `/opt/rustguac`, creates `rustguac` system user with home dir, sets up systemd services.
- **Docker**: `docker build -t rustguac .` — multi-stage, debian:trixie-slim runtime.
- **Remote test machine**: See project memory for connection details. Binary at `/opt/rustguac/bin/rustguac`, config at `/opt/rustguac/config.toml`.

## Build notes

- guacd is built from `../guacamole-server` (apache/guacamole-server)
- Debian 13 ships freerdp3-dev, not freerdp2-dev. guacamole-server 1.6.1+ has FreeRDP 3 auto-detection. Building with `--with-rdp`.
- **Patches required:** guacamole-server needs patches for FreeRDP 3.15+ (Debian 13). See `patches/README.md`. All build scripts apply these automatically.
- Chromium on headless VMs needs: `--in-process-gpu`, `--use-gl=angle`, `--use-angle=swiftshader`, `--disable-gpu-*`, `--disable-dev-shm-usage`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sol1/rustguac](https://github.com/sol1/rustguac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
