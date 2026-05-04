---
trigger: always_on
description: **Trusty** is a cross-platform Flutter GUI client for [Trusty VPN](https://github.com/TrustTunnel/TrustTunnelClient) (Apache 2.0).
---

# AGENTS.md — Trusty Project Guide

## Overview

**Trusty** is a cross-platform Flutter GUI client for [Trusty VPN](https://github.com/TrustTunnel/TrustTunnelClient) (Apache 2.0).
A wrapper around the CLI client: generates TOML configs and manages the process.

**Platforms:** Windows 10/11 (stable), macOS 11+ (alpha)

## Architecture

```
lib/
├── main.dart                      # Entry point, window/tray, navigation
├── models/
│   ├── server_config.dart         # VPN config model + TOML generation (toToml())
│   ├── server_setup_config.dart   # Remote server deployment config (SSH, domain, VPN creds)
│   ├── setup_step.dart            # Setup step enum with UI props
│   ├── vpn_status.dart            # Status enum with UI properties (color, icon, text)
│   └── domain_group.dart          # DomainGroup + DomainGroupsData models
├── services/
│   ├── config_service.dart        # Persistence (SharedPreferences) + TOML file I/O
│   ├── vpn_service.dart           # Process management, connection state, logs
│   ├── server_setup_service.dart  # Remote VPS deployment via SSH (dartssh2)
│   └── domain_discovery_service.dart  # HTTP fetch + HTML parse for related domains
└── screens/
    ├── home_screen.dart           # Connect/disconnect, status display
    ├── settings_screen.dart       # Server config form
    ├── server_setup_screen.dart   # Remote server deployment form + progress
    ├── split_tunnel_screen.dart   # Domain groups, app discovery, log suggestions
    └── logs_screen.dart           # Real-time log viewer
```

### State Management
- **Provider** + `ChangeNotifierProvider`
- `VpnService` — reactive connection state + logs
- `ServerSetupService` — SSH deployment state + logs
- `ConfigService` — plain service injected via `Provider`

### Connection Flow
1. User clicks Connect → `HomeScreen._handleButtonPress()`
2. `ConfigService.loadConfig()` → SharedPreferences
3. **macOS only:** `VpnService._ensureMacOSPrivileges(exePath)` — checks setuid bit via `stat -f %Sp`; if missing, calls `osascript` to show macOS password dialog and runs `chmod u+s` (one-time per install)
4. `ConfigService.writeConfigFile()` → TOML at `./client/trusttunnel_client.toml`
5. `VpnService.connect()` → spawns CLI with `--config` and `--loglevel`
6. stdout/stderr captured, parsed for log levels; consecutive similar lines collapsed via `_addLog()` deduplication
7. 2s stability check → Connected
8. Exit code != 0 → Error (Wintun check on Windows, TUN permission check on macOS)

### Log Deduplication
`_addLog()` in `vpn_service.dart` strips all digit sequences from the message to produce a pattern. Consecutive lines with the same pattern are collapsed in-place: `"System DNS proxy request id=14812 failed"` + 400 more → `"... request id=14812 failed (×401)"`. State resets on `clearLogs()` and on process exit.

### Server Deployment Flow (SSH)
1. User fills form → `ServerSetupService.installServer(config)`
2. Steps: SSH connect → check system → install → upload configs (SFTP) → certbot → systemd → verify
3. Each step updates `SetupStep` enum → UI rebuilds via `Consumer`
4. On success → "Apply to client" auto-fills settings

### Platform Abstraction

| Component | Windows | macOS |
|-----------|---------|-------|
| CLI binary | `trusttunnel_client.exe` | `trusttunnel_client` |
| Client dir | `Directory.current.path/client/` | Navigate up from `.app` bundle |
| TUN driver | Wintun (wintun.dll, 5s release wait) | Native utun + setuid via osascript (one-time) |
| Tray icon | `.ico` via `Platform.resolvedExecutable` path | `.png` via `.app/Contents/Frameworks/...` |
| App discovery | Program Files, AppData → `.exe` | `/Applications` → `.app` bundles |

Key platform checks in code:
- `config_service.dart`: `getClientDirectory()`, `getTrustTunnelExecutable()`
- `vpn_service.dart`: `_ensureMacOSPrivileges()` + `_hasMacOSSetuid()` (macOS); Wintun waits in `connect()`, `disconnect()`, `shutdown()` (Windows, 4 places)
- `split_tunnel_screen.dart`: `_getInstalledAppsWindows()` / `_getInstalledAppsMacOS()`
- `main.dart`: tray icon path
- `home_screen.dart`: help text (exe name)
- `server_setup_screen.dart`: SSH key path hint

## File Locations at Runtime

| File | Windows | macOS |
|------|---------|-------|
| CLI binary | `./client/trusttunnel_client.exe` | `../client/trusttunnel_client` (relative to .app) |
| TUN driver | `./client/wintun.dll` | Not needed |
| Config | `./client/trusttunnel_client.toml` | `../client/trusttunnel_client.toml` |
| Tray icon | `./data/flutter_assets/assets/tray_icon.ico` | Inside `.app` bundle (`.png`) |

## CI/CD

Two GitHub Actions workflows triggered on `v*` tags:

- `.github/workflows/release.yml` — Windows (stable release)
- `.github/workflows/release-macos.yml` — macOS (alpha prerelease)

Both download the **latest** CLI from [TrustTunnelClient releases](https://github.com/TrustTunnel/TrustTunnelClient/releases) at build time.

### Release Process
1. Update version in `pubspec.yaml`
2. `git commit && git tag vX.Y.Z && git push origin vX.Y.Z`
3. Both workflows run automatically

## Important Rules

### Security
- **NEVER** commit real credentials
- `.gitignore` blocks `*.toml` (except `.toml.example`), `*.exe`, `wintun.dll`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meddelin/trusty](https://github.com/Meddelin/trusty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
