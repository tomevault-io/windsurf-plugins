---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# ⚠️ BUILD ORDER: web FIRST, then backend.
# The Go binary embeds web/ via //go:embed, so the web build must exist
# before go build runs. `make all` enforces this; plain go build does not.

# Step 1: Build Lit web UI
# ⚠️ MUST use `make web` — NEVER run `npx vite build` directly.
# The Makefile handles conditional rebuild (stamp-based) and cleanup.
make web

# Step 2: Build Go backend for current platform (embeds web/)
go build -o wisper .

# Or build all platforms (linux, darwin, windows) — runs web + go build
make all

# Build Go backend for a specific target (from Makefile)
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o dist/linux-amd64/wisper .

# Force web rebuild (ignore stamp)
make web-force

# Run the server (defaults to :8900)
./wisper
./wisper -addr :9000                    # custom port
./wisper -version                       # print version and exit

# Run Go tests
go test ./... -v
go test ./api/ -v -run TestListTunnels  # single test suite
```

## Architecture

Wisper is a **GOST tunnel manager** — a Go HTTP API server with an embedded Lit web UI for creating and managing reverse proxy tunnels through the GOST network.

The project is a **single Go module** (`github.com/go-gost/wisper`, `go.mod` at root). The Lit web app lives under `web-src/` and is embedded into the Go binary via `//go:embed` in `web.go`.

### How the two halves connect

1. `npx vite build` in `web-src/` outputs to `web/`
2. `web.go` embeds `web/*` via `//go:embed` and serves it as an SPA (non-file requests → `index.html`)
3. `api/server.go` registers API routes on `/api/*` and falls back to the web handler for everything else
4. The `GoBackend` TypeScript class uses relative URLs when base is empty, so it works same-origin

### Go package layout

| Package | Purpose |
|---------|---------|
| `main` (`main.go`) | Entry point: parses flags, inits config, starts stats runner, starts HTTP server, graceful shutdown |
| `web.go` | Embeds Lit web build and serves it with SPA fallback |
| `config/` | App settings + tunnel/entrypoint persistence to `~/.config/wisper/config.yml`. Thread-safe via `atomic.Value` with deep-copy semantics |
| `tunnel/` | `Tunnel` interface + 4 concrete types (file, http, tcp, udp) + `ChainConfig` builder |
| `tunnel/entrypoint/` | Entrypoint types (tcp, udp) implementing `tunnel.Tunnel` interface |
| `api/` | REST handlers (`Go 1.22 ServeMux` with method routing) + CORS middleware |
| `runner/` | Background task scheduler: async execution, optional repeat interval, cancel-by-ID |
| `runner/task/` | Concrete tasks — currently only `stats` polling |
| `version/` | Version string (set via `-ldflags="-X main.version=..."` at link time) |

### Startup flow

1. `main()` parses `-addr` and `-version` flags
2. `config.Init()` creates `~/.config/wisper/`, loads `config.yml` (creates empty config if missing), initializes structured logging
3. `tunnel.LoadConfig()` and `entrypoint.LoadConfig()` reconstruct tunnel/entrypoint objects from persisted config, auto-starting non-closed ones
4. `runner.Exec()` starts the stats polling task (1s interval, async)
5. HTTP server starts on the configured address with the combined API + web handler
6. On SIGINT/SIGTERM: persist state (`tunnel.SaveConfig()` + `entrypoint.SaveConfig()`), graceful HTTP shutdown (5s timeout)

### Tunnel types and their internal structure

Every tunnel type follows the same pattern:
- **Constructor** (`NewFileTunnel`, `NewHTTPTunnel`, etc.): generates a UUID ID → MD5 hash for the public endpoint subdomain, sets defaults
- **`init()`**: builds `x/config.ServiceConfig` structs describing the GOST listener + handler + chain
- **`Run()`**: parses the chain config, instantiates the GOST listener/handler/router/hop objects, wires them together, starts `Serve()` in a goroutine
- **`Close()`** / **`IsClosed()`**: close-once channel pattern

**File tunnel** (`tunnel/file.go`): Two-phase setup — starts a local TCP file server (`:0`), then a reverse-TCP forwarder (rtcp) that tunnels through WSS to `tunnel.gost.run:443`. The forwarder's hop targets the local file server address.

**HTTP tunnel** (`tunnel/http.go`): Single reverse-TCP forwarder (rtcp) that tunnels through WSS and forwards to a local HTTP endpoint. Supports TLS to backend, host rewriting, sniffing, basic auth.

**TCP tunnel** (`tunnel/tcp.go`): Reverse-TCP forwarder through WSS to a raw TCP endpoint.

**UDP tunnel** (`tunnel/udp.go`): Reverse-UDP forwarder (rudp) through WSS to a raw UDP endpoint.

**TCP entrypoint** (`tunnel/entrypoint/tcp.go`): Local TCP listener (`handler: tcp`) that forwards through the tunnel chain. This is the reverse of a tunnel — it listens locally and sends traffic *out* through the GOST tunnel.

**UDP entrypoint** (`tunnel/entrypoint/udp.go`): Same as TCP entrypoint but for UDP, with keepalive+TTL support.

### Key dependency: `github.com/go-gost/x`

The `x` module (declared as a versioned module in `go.mod`) provides concrete GOST implementations. Wisper uses:
- Listeners: `tcp`, `rtcp` (reverse-tcp), `rudp` (reverse-udp), `udp`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-gost/wisper](https://github.com/go-gost/wisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
