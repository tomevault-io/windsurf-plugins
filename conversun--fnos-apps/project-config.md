---
trigger: always_on
description: **Generated:** 2026-02-13
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-13
**Commit:** 4005051
**Branch:** main

## OVERVIEW

Monorepo packaging 18 third-party apps as `.fpk` installers for fnOS NAS. Pure bash — downloads upstream binaries, merges with shared lifecycle framework, outputs `.fpk` tarballs. Daily CI auto-syncs upstream versions.

## STRUCTURE

```
fnos-apps/
├── shared/              # Shared lifecycle framework (all apps inherit)
│   ├── cmd/             # Daemon mgmt, install/upgrade/uninstall hooks (see shared/cmd/AGENTS.md)
│   └── wizard/          # Default uninstall wizard (JSON)
├── apps/
│   ├── plex/            # Plex: port 32400, downloads .deb from plex.tv API
│   ├── emby/            # Emby: port 8096, downloads .deb from GitHub Releases
│   ├── jellyfin/        # Jellyfin: port 8097, free media system
│   ├── qbittorrent/     # qBittorrent: port 8085, downloads static binary. Most complex app.
│   ├── gopeed/          # Gopeed: port 9999, multi-protocol downloader
│   ├── ani-rss/         # ANI-RSS: port 7789, anime RSS auto-download
│   ├── audiobookshelf/  # Audiobookshelf: port 13378, audiobook/podcast server
│   ├── nginx/           # Nginx: port 8888, reverse proxy and HTTP server
│   ├── certimate/       # Certimate: port 8090, Go binary, SSL certificate manager
│   ├── syncthing/       # Syncthing: port 8384, Go binary, P2P file sync
│   ├── navidrome/       # Navidrome: port 4533, Go binary, music streaming
│   ├── sun-panel/       # Sun-Panel: port 3002, Go+Vue binary, NAS dashboard
│   ├── openlist/        # OpenList: port 5244, Go binary, file list/WebDAV
│   ├── vaultwarden/     # Vaultwarden: port 8880, Rust binary, password manager
│   ├── moviepilot/      # MoviePilot: port 3000, Python app, requires python312 runtime
│   ├── kavita/          # Kavita: port 5000, .NET self-contained binary
│   └── tinymediamanager/# tinyMediaManager: port 5800, Java app
├── scripts/
│   ├── build-fpk.sh     # Generic fpk packager (shared + app-specific → .fpk)
│   ├── new-app.sh       # Scaffold new app: ./scripts/new-app.sh <name> <display> <port>
│   ├── apps/            # Per-app build contracts (meta.env, build.sh, get-latest-version.sh, release-notes.tpl)
│   │   ├── plex/
│   │   ├── emby/
│   │   ├── jellyfin/
│   │   ├── qbittorrent/
│   │   ├── gopeed/
│   │   ├── ani-rss/
│   │   ├── audiobookshelf/
│   │   └── nginx/
│   ├── lib/             # Shared build utilities
│   │   └── update-common.sh  # Common functions for app builds
│   └── ci/              # CI helper scripts
│       └── resolve-release-tag.sh  # Version tag resolution with -r2/-r3 auto-increment
└── .github/workflows/   # CI: single dynamic workflow + reusable build pipeline
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add new app | `scripts/new-app.sh` | Generates full skeleton with TODO markers |
| Understand app lifecycle | `shared/cmd/common` | 352-line core: daemon start/stop, install hooks, logging |
| Service entry point | `shared/cmd/main` | start/stop/status dispatcher, sources `common` + `service-setup` |
| App-specific service config | `apps/*/fnos/cmd/service-setup` | Sets SERVICE_COMMAND, PID/LOG paths |
| App-specific overrides | `apps/*/fnos/cmd/` | Files here overlay shared/cmd/ in fpk |
| Build locally | `apps/*/update_*.sh` | Downloads upstream, builds app.tgz + fpk |
| App build contract | `scripts/apps/<app>/` | meta.env, build.sh, get-latest-version.sh, release-notes.tpl |
| Build utilities | `scripts/lib/update-common.sh` | Shared functions: info(), warn(), error(), cleanup trap |
| Version tag resolution | `scripts/ci/resolve-release-tag.sh` | Determines release tag with -r2/-r3 auto-increment |
| Generic fpk packager | `scripts/build-fpk.sh` | Unified packaging: shared + app-specific → .fpk |
| CI/CD entry | `.github/workflows/build-apps.yml` | Single workflow: detect changed apps → dynamic matrix build |
| CI/CD pipeline | `.github/workflows/reusable-build-app.yml` | Reusable workflow: check-version → build (x86+arm) → release |
| App metadata | `apps/*/fnos/manifest` | Key=value: appname, version, port, checksum |
| User/group permissions | `apps/*/fnos/config/privilege` | JSON: run-as user, extra groups (video/render for Plex) |
| Port forwarding rules | `apps/*/fnos/*.sc` | fnOS firewall port config |
| Desktop UI entry | `apps/*/fnos/ui/config` | JSON: app launcher URL, port, icon paths |
| Icons | `apps/*/fnos/ICON*.PNG` + `ui/images/` | ICON.PNG (small) + ICON_256.PNG (large) |

## CONVENTIONS

- **Language**: 100% bash. No package managers, no compiled languages.
- **Manifest format**: Fixed-width alignment at column 16 (`appname         = value`).
- **fpk = tar.gz** containing: `app.tgz` + `cmd/` + `config/` + `wizard/` + `manifest` + icons + UI.
- **Overlay pattern**: `shared/cmd/*` copied first, then `apps/*/fnos/cmd/*` overwrites. App-specific wins.
- **Architecture**: Always dual-build x86 + arm. Detect via `uname -m`.
- **Version tags**: Namespaced — `plex/v1.42.2.10156`, `qbittorrent/v5.1.4-r2`.
- **Revision suffix**: `-r2`, `-r3` auto-incremented for same-version re-releases.
- **Scripts use Chinese** for user-facing messages (info/warn/error), English for code comments.
- **Color output**: `info()` green, `warn()` yellow, `error()` red+exit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conversun/fnos-apps](https://github.com/conversun/fnos-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
