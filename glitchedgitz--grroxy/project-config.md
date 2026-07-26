---
trigger: always_on
description: Cybersecurity proxy toolkit that blends manual web testing with AI agents. Intercepts HTTP/HTTPS traffic, provides request modification, fuzzing, browser automation, and an MCP endpoint for AI-driven security analysis. Built as a multi-binary Go backend with a Svelte frontend.
---

# Grroxy

Cybersecurity proxy toolkit that blends manual web testing with AI agents. Intercepts HTTP/HTTPS traffic, provides request modification, fuzzing, browser automation, and an MCP endpoint for AI-driven security analysis. Built as a multi-binary Go backend with a Svelte frontend.

**Version:** 2026.4.2 (App) / 0.29.1 (Backend & Frontend)

## Tech Stack

- **Backend:** Go 1.24 (toolchain go1.24.1)
- **Database:** PocketBase (forked: `github.com/glitchedgitz/pocketbase`)
- **Frontend:** Svelte 5 + Vite + Tailwind CSS + SvelteKit (dev UI)
- **Desktop:** Electron 36.4.0
- **CLI:** cobra (command framework)
- **HTTP:** echo/v5 (web framework), custom raw HTTP parser, uTLS
- **Browser:** chromedp (Chrome DevTools Protocol)
- **Key libs:** cook/v2 (payload generation), dadql (query language), wappalyzergo (tech detection), mcp-go (Model Context Protocol), fsnotify (file watching)

## Architecture

### Binaries

| Binary        | Entry Point              | Purpose                                                                        |
| ------------- | ------------------------ | ------------------------------------------------------------------------------ |
| `grroxy`      | `cmd/grroxy/main.go`     | **Launcher** - manages projects, starts per-project backends, global templates |
| `grroxy-app`  | `cmd/grroxy-app/main.go` | **Project backend** - proxy, intercept, templates, all per-project APIs        |
| `grroxy-tool` | `cmd/grroxy-tool/`       | Standalone tools server (fuzzer, SDK)                                          |
| `grx-fuzzer`  | `cmd/grx-fuzzer/main.go` | Standalone HTTP/HTTP2 fuzzer CLI                                               |
| `grxp`        | `cmd/grxp/main.go`       | URL parser/prober from stdin with dadql filtering                              |

### Directory Layout

```
apps/
  app/           # grroxy-app backend logic (~40 files: proxy, intercept, templates, MCP, etc.)
  launcher/      # grroxy launcher backend (project management, template distribution)
  tools/         # grroxy-tool backend (fuzzer)
cmd/
  grroxy/        # Main CLI entry + migrations
  grroxy-app/    # Project app entry + migrations
  grroxy-tool/   # Tool server entry + migrations
  electron/      # Electron desktop wrapper
  grx-fuzzer/    # Fuzzer CLI
  grxp/          # URL parser CLI
grx/
  browser/       # Chrome automation (chromedp wrappers)
  dev/           # Developer UI (SvelteKit, served at /dev)
  frontend/      # Main production frontend (Svelte, bundled into binary)
  fuzzer/        # Fuzzing engine (cluster bomb, pitch fork modes)
  rawhttp/       # Raw HTTP/1.1 & HTTP/2 parser and client
  rawproxy/      # MITM proxy with TLS, cert generation, WebSocket support
  templates/     # Template engine with hooks and default configs
  version/       # Version constants
internal/
  config/        # Config struct and initialization
  logflags/      # Log flag setup
  process/       # Process/command management
  save/          # File saving utilities
  schemas/       # PocketBase collection schemas (22 files)
  sdk/           # Client SDK
  types/         # Shared type definitions
  updater/       # Binary self-updater (GitHub Releases)
  utils/         # Utility functions
```

### Data Flow

1. `grroxy start` boots the **launcher** (default `127.0.0.1:8090`), manages projects via PocketBase
2. Opening a project spawns a **`grroxy-app`** subprocess with `--host`, `--path`, `--launcher` flags
3. `grroxy-app` starts its own PocketBase instance in the project directory, boots proxy listeners
4. Proxy intercepts HTTP/HTTPS traffic via MITM, stores requests/responses in PocketBase collections
5. Frontend communicates with both launcher and project APIs
6. Templates with hooks (`on_request`, `on_response`, `on_new_sitemap`) run automatically on proxy traffic

### Config Directories

- **Config:** `~/.config/grroxy/` (CA certs, config)
- **Projects:** `$XDG_CONFIG_DIR/grroxy/` (project databases, launcher DB in `grroxy-main/`)
- **Cache:** `$XDG_CACHE_DIR/grroxy/`
- **Templates:** configurable via `GRROXY_TEMPLATE_DIR` env var

## Build & Run

### Backend

```bash
# Build all binaries
go build ./cmd/grroxy
go build ./cmd/grroxy-app
go build ./cmd/grroxy-tool

# Run launcher
./grroxy start
./grroxy start --host 127.0.0.1:9090  # custom port

# Run project app directly (normally spawned by launcher)
./grroxy-app --host 127.0.0.1:8091 --path /path/to/project --launcher 127.0.0.1:8090

# Run migrations
./grroxy migrate up
```

### Frontend (Dev UI)

```bash
cd grx/dev
npm install
npm run dev       # Dev server
npm run build     # Production build
npm run check     # TypeScript + Svelte check
```

### Frontend (Main)

```bash
cd grx/frontend
npm install
npm run dev
npm run build
```

### Electron Desktop

```bash
cd cmd/electron
npm install
npm start         # Dev mode
npm run build     # Build installers (DMG/NSIS/AppImage)
```

### Release (cross-compile all platforms)

```bash
./release.sh  # Builds darwin/arm64, darwin/amd64, linux/amd64, linux/arm64, windows/amd64
```

## CLI Flags


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glitchedgitz/grroxy](https://github.com/glitchedgitz/grroxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
