---
trigger: always_on
description: Voyager (Client) <-> [WebSocket] <-> Wormhole (Relay) <-> [WebSocket] <-> Horizon (Host) <-> PTY/Shell
---

# AGENTS.md

## Architecture
```
Voyager (Client) <-> [WebSocket] <-> Wormhole (Relay) <-> [WebSocket] <-> Horizon (Host) <-> PTY/Shell
```

### Network Modes
1. **LAN Mode**: Direct WebSocket (lowest latency)
2. **WAN Mode**: Through Wormhole relay (NAT traversal)
3. **WireGuard Direct**: UDP hole-punched (bypasses relay)

## Project Structure
```
Blackhole/
├── horizon/                 # Host terminal server (Flutter)
│   ├── lib/src/             # Dart app logic
│   ├── daemon/src/          # Rust daemon (WG, TUN, NAT, DNS)
│   ├── macos/Runner/        # macOS native (PTY, VPN helper)
│   ├── linux/runner/        # Linux native (PTY)
│   └── windows/runner/      # Windows native (ConPTY)
├── voyager/                 # Remote terminal client (Flutter)
│   ├── lib/src/             # Dart app logic
│   ├── ios/VoyagerTunnel/   # iOS Network Extension
│   ├── macos/VoyagerTunnel/ # macOS Network Extension
│   └── voyager_share/       # Shared widgets (HHKB, UI)
├── wormhole/                # Relay server (Rust)
├── tunnel/                  # WireGuard tunnel lib (Rust, iOS/macOS)
├── internal/                # CI/CD (GitHub Actions, Dockerfiles)
└── xterm/                   # Forked xterm terminal widget
```

## Development Commands
```bash
cd horizon && flutter run -d macos        # Horizon
cd voyager && flutter run -d ios          # Voyager
cd wormhole && WORMHOLE_TOKEN=xxx cargo run  # Wormhole
flutter test                              # Tests
dart format . && flutter analyze          # Lint
cargo fmt && cargo clippy                 # Rust lint
```

## CI/CD and Deployment

### Build Pipeline
The **internal** repo (`internal/`) contains GitHub Actions workflow.
**Trigger**: Push to `main` of `Blackhole-internal`.

```bash
cd internal && git add -A && git commit -m "trigger build" && git push origin main
```

| Artifact | Destination |
|----------|-------------|
| `ghcr.io/icyoung/blackhole-landing:latest` | GHCR |
| `ghcr.io/icyoung/blackhole-voyager:latest` | GHCR |
| `ghcr.io/icyoung/blackhole-wormhole:latest` | GHCR |
| `horizon-macos.dmg` (signed + notarized) | R2 |
| `horizon-linux.tar.gz`, `horizon-windows.zip` | R2 |
| `voyager-macos.dmg` (signed + notarized) | R2 |
| `voyager-linux.tar.gz`, `voyager-windows.zip` | R2 |
| `voyager-android.apk` | R2 |

Downloads: `https://download.blackhole-ai.com/<artifact>`

### Production Server (lightnode)
**Host**: `38.60.162.209` (SSH: `lightnode`)
**Compose**: `/opt/blackhole/deploy/docker-compose.yml`

| Service | Domain | Image |
|---------|--------|-------|
| Traefik | — | `traefik:v2.10` (reverse proxy + TLS) |
| Landing | `blackhole-ai.com` | `ghcr.io/icyoung/blackhole-landing:latest` |
| Voyager Web | `app.blackhole-ai.com` | `ghcr.io/icyoung/blackhole-voyager:latest` |
| Wormhole | `wormhole.blackhole-ai.com` | `blackhole-wormhole:local` |

Wormhole built from `/opt/blackhole/deploy/wormhole-src/`. UDP 6666 for netcheck.

**Deploy**:
```bash
# Landing + Voyager Web (pull GHCR images)
ssh lightnode "cd /opt/blackhole/deploy && docker compose pull landing voyager-web && docker compose up -d landing voyager-web"

# Wormhole (rebuild from source)
ssh lightnode "cd /opt/blackhole/deploy && docker compose build --no-cache wormhole && docker compose up -d wormhole"
```

### Repos
| Repo | URL |
|------|-----|
| Blackhole (public) | `git@github.com:Icyoung/Blackhole.git` |
| Blackhole-internal | `git@github.com:Icyoung/Blackhole-internal.git` |

## Commit Convention
Conventional commits: `feat(horizon):`, `fix(voyager):`, `chore:`, etc.

## Environment Variables
- `WORMHOLE_TOKEN` - Auth token (required)
- `PORT` - Server port (default: 6666)
- `WORMHOLE_NETCHECK_HOST` / `WORMHOLE_NETCHECK_PORT` - UDP netcheck

<!-- ufoo -->
## ufoo Protocol

This project uses **ufoo** for agent coordination. Read the full documentation at `.ufoo/docs/` (symlinked from ufoo installation).

### Core Principles

1. **Agents are autonomous** - Execute tasks without asking for permission
2. **Communication via bus** - Use `ufoo bus` for inter-agent messaging
3. **Decisions are recorded** - Use `ufoo ctx` for decision tracking
4. **Context is shared** - All agents read from `.ufoo/context/`

### Available Commands

| Command | Description |
|---------|-------------|
| `uinit` | Initialize/repair .ufoo directory |
| `uctx` | Check context status and decisions |
| `ustatus` | Unified status view (banner, unread bus, open decisions) |
| `ubus` | Check bus messages and **auto-execute** them |

### Quick Reference

```bash
# Context
ufoo ctx decisions -l          # List all decisions
ufoo ctx decisions -n 1        # Show latest decision

# Bus
ufoo bus join                  # Join bus (auto by uclaude/ucodex)
ufoo bus check $SUBSCRIBER     # Check pending messages
ufoo bus send "<id>" "<msg>"   # Send message
ufoo bus status                # Show bus status
```

---

## ufoo context Protocol

On session start, check context status:
```bash
ufoo ctx decisions -l
ufoo ctx decisions -n 1
```

Key files in `.ufoo/context/`:
- `DECISIONS/` - Decision log (append-only)
- `SYSTEM.md` - System overview
- `CONSTRAINTS.md` - Non-negotiable rules

**Decision recording policy:**
- **Must record**: evaluations, architecture, naming, trade-offs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Icyoung/Blackhole](https://github.com/Icyoung/Blackhole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
