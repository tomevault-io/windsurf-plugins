---
trigger: always_on
description: TSDProxy — Go reverse proxy that auto-exposes Docker containers via Tailscale. Labels Docker containers with `tsdproxy.*` to create per-container Tailscale machines with automatic HTTPS. Stack: Go 1.26, templ (UI), Vite/Bun (frontend), Hugo (docs), zerolog (logging).
---

# PROJECT KNOWLEDGE BASE

## OVERVIEW

TSDProxy — Go reverse proxy that auto-exposes Docker containers via Tailscale. Labels Docker containers with `tsdproxy.*` to create per-container Tailscale machines with automatic HTTPS. Stack: Go 1.26, templ (UI), Vite/Bun (frontend), Hugo (docs), zerolog (logging).

## STRUCTURE

```
tsdproxy/
├── cmd/
│   ├── server/main.go          # Main server binary (WebApp, InitializeApp)
│   └── healthcheck/main.go     # Docker HEALTHCHECK binary (GET /health/ready/)
├── internal/
│   ├── api/                    # REST API routes (JSON endpoints)
│   ├── config/                 # Config loading, validation, fsnotify file watching
│   ├── consts/                 # Shared constants (headers, proxy manager keys)
│   ├── core/                   # HTTP server, logging, health, sessions, CSRF, version, telemetry
│   │   ├── metrics/            # Prometheus-style metrics
│   │   └── webhook/            # Webhook dispatch on proxy events
│   ├── dashboard/              # SSE dashboard routes + streaming + preferences + API
│   ├── dnsproviders/           # DNS Provider interface + Cloudflare/MagicDNS implementations
│   ├── dom/                    # ID generation utility
│   ├── model/                  # Shared types: Config, PortConfig, ProxyStatus, events
│   ├── proxymanager/           # Central orchestrator: wires target→proxy→DNS→TLS providers
│   ├── proxyproviders/         # ProxyProvider interface + Tailscale (per-proxy & shared)
│   │   └── tailscale/          # Tailscale provider: Proxy, SharedProxy, SharedServer, SNIRouter
│   ├── targetproviders/        # TargetProvider interface + Docker/List implementations
│   │   ├── docker/             # Docker label parsing, container resolution, port mapping
│   │   └── list/               # Static YAML file-based target provider
│   ├── tlsproviders/           # TLS Provider interface + ACME/Tailscale implementations
│   └── ui/                     # templ server-rendered components (proxy cards, pages, layouts)
├── web/                        # Frontend: Vite/Bun + htmx, go:embed dist via statigz+brotli
├── docs/                       # Hugo docs site (separate go.mod: github.com/imfing/hextra-starter-template)
├── dev/                        # Dev docker-compose configs + sample tsdproxy.yaml + data
├── e2e/                        # E2E tests (//go:build e2e, testcontainers + real Tailscale)
└── contrib/                    # Community templates (Unraid)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add a new target provider | `internal/targetproviders/` | Implement `TargetProvider` (6 methods) |
| Add a new proxy provider | `internal/proxyproviders/` | Implement `Provider` + `ProxyInterface` (+ optional `RawTCPListener`, `DomainRequiredProvider`) |
| Add a new DNS provider | `internal/dnsproviders/` | Implement `Provider` (4 methods); for ACME also implement `certmagic.DNSProvider` |
| Add a new TLS provider | `internal/tlsproviders/` | Implement `Provider` (4 methods) |
| Change Docker label parsing | `internal/targetproviders/docker/consts.go` | All label constants (`tsdproxy.*`) |
| Change port mapping logic | `internal/targetproviders/docker/container.go` | `getPorts()`, `getTargetURL()` |
| Modify dashboard UI | `internal/ui/pages/proxylist.templ` | templ template for proxy cards |
| Add frontend assets | `web/` | Build with `bun run build`, embedded via go:embed |
| Change config format | `internal/config/config.go` | Struct definitions; `configfile.go` for I/O |
| Add HTTP routes | `internal/dashboard/dash.go` | `AddRoutes()` method |
| Change logging | `internal/core/log.go` | zerolog setup + HTTP middleware |
| Change release process | `.goreleaser.yaml` | Multi-arch Docker, version embedding |
| Tailscale auth flow | `internal/proxyproviders/tailscale/provider.go` | OAuth vs AuthKey resolution |
| Shared Tailscale mode | `internal/proxyproviders/tailscale/shared_server.go` | Ref-counted tsnet.Server, SNI routing |
| DNS record management | `internal/dnsproviders/` | LifecycleManager wraps create/delete/validate with retry |
| TLS certificate provisioning | `internal/tlsproviders/` | LifecycleManager wraps provision/cleanup |
| Add E2E tests | `e2e/` | `//go:build e2e`, real tsdproxy binary + Tailscale + testcontainers |
| Wire new provider into orchestrator | `internal/proxymanager/proxymanager.go` | Add case in `add*Providers()` switch |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `WebApp` | Struct | `cmd/server/main.go` | Root app container, owns all subsystems |
| `InitializeApp` | Func | `cmd/server/main.go` | Bootstrap: config→logger→HTTP→health→proxy→dashboard |
| `TargetProvider` | Interface | `internal/targetproviders/targetproviders.go` | 6-method contract: WatchEvents, AddTarget, DeleteProxy, ReResolve, Close |
| `Provider` | Interface | `internal/proxyproviders/proxyproviders.go` | Factory: ResolveAuthKey + NewProxy |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [almeidapaulopt/tsdproxy](https://github.com/almeidapaulopt/tsdproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
