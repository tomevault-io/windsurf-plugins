---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project overview

**Trayscale** is an unofficial GUI for the Tailscale daemon, aimed primarily at Linux (no official Linux GUI client exists). It provides a system tray icon and a Libadwaita-based window for managing Tailscale features such as peers, exit nodes, Taildrop, profiles, and Mullvad exit nodes.

The app talks to the local Tailscale daemon (`tailscaled`) via the Tailscale local API and CLI helpers from the `tailscale.com` module. Full functionality requires the current user to be configured as the Tailscale operator (`sudo tailscale set --operator=$USER`).

## Technology stack

| Layer | Choice |
|-------|--------|
| Language | Go — see `go.mod` for the required toolchain |
| GUI | GTK 4 + Libadwaita via `github.com/diamondburned/gotk4` and `gotk4-adwaita` |
| Tray | `deedles.dev/tray` (StatusNotifierItem); not maintained on macOS |
| Tailscale | `tailscale.com` local client / IPN APIs |
| UI design | Cambalache (`.cmb` project + per-window `.ui` XML) |
| Settings | GSettings schema `dev.deedles.Trayscale` |
| App ID | `dev.deedles.Trayscale` |

This file should not pin toolchain or dependency versions (they go stale). Prefer “as specified in `go.mod`” (or the README for system libraries). Other project docs, such as the README, may name specific versions when useful.

## Directory structure

```
.
├── cmd/trayscale/           # Main binary entrypoint
│   ├── trayscale.go        # main(), optional PPROF CPU profiling
│   └── default.pgo         # Profile-guided optimization profile
├── internal/
│   ├── ui/                 # Application UI (gotk4/adwaita)
│   │   ├── app.go          # App lifecycle, notifications, updates
│   │   ├── mainwindow.go   # Primary window and peer stack
│   │   ├── *page.go        # Per-peer / offline / Mullvad pages
│   │   ├── *.ui            # GtkBuilder XML (edit via Cambalache)
│   │   ├── trayscale.cmb   # Cambalache project for the .ui files
│   │   ├── app.css         # Application CSS
│   │   └── ...
│   ├── tsutil/             # Tailscale client helpers and status poller
│   ├── tray/               # System tray icon and menu
│   ├── gutil/              # GTK helpers (FillFromBuilder/UI, CSS, widgets)
│   ├── listmodels/         # gio.ListModel iterators and binding helpers
│   ├── giofs/              # gio.File → io/fs bridge (file pickers / Taildrop)
│   └── metadata/           # App ID, version, license, release notes
├── assets.go               # Embedded LICENSE + metainfo (package trayscale)
├── dist.sh                 # Packaging helper (build / install / macOS)
├── dev.deedles.Trayscale.* # Desktop entry, icon, GSettings schema, AppStream metainfo
├── gschemas.compiled       # Local compiled schemas (gitignored; regenerate as needed)
└── go.mod
```

- Application code lives under `internal/`. The only public package is the module root (`assets.go`).
- UI logic is Go; layouts are GtkBuilder XML loaded with `//go:embed` and `gutil.FillFromUI`.

## Development commands

System packages needed for development (names vary by distro): GTK 4, Libadwaita, and GObject introspection development headers. CI installs the equivalent of `libgtk-4-dev`, `libadwaita-1-dev`, and `gobject-introspection` on Ubuntu.

```bash
# Download modules
go mod download

# Run (compiles as needed)
go run ./cmd/trayscale
# Optional: start with the main window hidden
go run ./cmd/trayscale --hide-window

# Tests, vet, format — go test already compiles packages; a separate
# go build is unnecessary for verification
go test ./...
go vet ./...
go fmt ./...

# Produce a binary only when you need one (not for routine checks)
go build -o trayscale ./cmd/trayscale
# Or with version injection (uses git describe if version omitted)
./dist.sh build [version]

# Staticcheck (declared as a tool in go.mod)
go tool staticcheck ./...

# Validate AppStream metainfo
appstreamcli validate --pedantic --no-net --explain dev.deedles.Trayscale.metainfo.xml
```

For local GSettings during development, compile the schema into the repo root (or install it system-wide / under `$XDG_DATA_DIRS`):

```bash
glib-compile-schemas .
# gschemas.compiled is gitignored
```

### Environment variables

| Variable | Purpose |
|----------|---------|
| `PPROF` | If set to a file path, write a CPU profile on exit |
| `TRAYSCALE_PRIVATE` | If `1`, enables private-mode behavior in `internal/metadata` |

## Packaging and distribution

- **`dist.sh build [version]`** — builds `./trayscale` with `-trimpath` and injects version via `-ldflags` into `internal/metadata.version`.
- **`dist.sh install <destdir>`** — installs binary, icon, desktop file, metainfo, and GSettings schema into a FHS-like tree.
- **`dist.sh install-macos` / `uninstall-macos`** — Homebrew-prefix install helpers (community/unofficial path).
- Flathub and AUR packages are maintained outside this repo; keep `dev.deedles.Trayscale.metainfo.xml` release notes accurate when shipping versions.

## Architecture notes

```
tailscaled (local API)
        ▲
        │
   internal/tsutil
   (Client, Poller, status types)
        │
        ▼
   internal/ui.App  ──►  MainWindow + Page stack
        │                     ├── SelfPage
        │                     ├── PeerPage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeedleFake/trayscale](https://github.com/DeedleFake/trayscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
