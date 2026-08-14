---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClassicStack is a Go-based AppleTalk Phase 2 router and AFP file server. It also supports other legacy protocols such as NetBEUI, NetBIOS and SMB. 
It bridges legacy Apple networking protocols to modern environments, supporting EtherTalk (raw Ethernet), LToUDP (multicast UDP), TashTalk (serial), and virtual LocalTalk transports.

**Module:** `github.com/ObsoleteMadness/ClassicStack`  
**Go version:** 1.23.0

## Remember!
1. Always confirm implementation details with the specifications found in /spec/*.md
2. Use consts rather than hard-coded values, especially for responses, errors, etc. 
3. Use the names from the specification for functions, consts, etc and include a comment with a breif description from the spec for any functions.
4. Captures of protocols can be found in /captures. Use `tshark` to review protocol captures to aid in diagnosing faults. 
5. When the observation from a capture differs from the spec, document it in the code and in `/spec/errata.md`
6. Where we do not have a spec and implementation is from observation, add details on wire format, observed commands, observed responses. Eg, the MacIPX Gateway implementation will be based on observed IPX encapsulation over AppleTalk traffic between a Novell Server and a Macintosh MacIPX client.
7. If code is from 3rd parties, **Always** attribute it to the original authors. 
8. Check for linting errors before committing.
9. Run gofmt before commiting.


## Commands

```bash
# Build
go build -tags all -o classicstack ./cmd/classicstack

# Run all tests
go test ./...

# Run tests for a specific package
go test ./service/afp/...

# Run with TOML config
./classicstack  # auto-loads server.toml if present

# Run with flags (see README.md for full list)
./classicstack -ethertalk eth0 -zone "MyZone"
```

## Architecture

### Core Data Flow

```
cmd/classicstack/main.go  →  internal/app (run-core)  →  Ports  →  Router  →  Services
```

1. **Entry point** (`cmd/classicstack/`) is a thin `main()` that calls `internal/app`, which parses CLI flags and `server.toml`, constructs ports, wires them to the router, and starts services. Two sibling commands wrap the same run-core for background operation: `cmd/classicstack-svc` (Windows service) and `cmd/classicstackd` (Unix/macOS daemon).
2. **Router** (`router/`) receives DDP datagrams from all ports, maintains the `RoutingTable` and `ZoneInformationTable`, and dispatches to services by socket number or forwards to other ports.
3. **Ports** (`port/`) abstract network interfaces. All implement `port.Port` (Unicast/Broadcast/Multicast). Implementations: `ethertalk`, `localtalk/ltoudp`, `localtalk/tashtalk`, `localtalk/virtual`.
4. **Services** (`service/`) plug into the router by registering socket numbers. Each implements `service.Service`.

### Key Packages

| Package | Role |
|---|---|
| `internal/app/` | The run-core (formerly `cmd/classicstack` package `main`): flag/TOML parsing, the `Supervisor`, every `wireXxx` hook, control-plane + web UI wiring. Exposes `Main(Version)` and `Run(ctx, args, Version)` so the interactive binary and the service/daemon wrappers all share one runtime. |
| `cmd/classicstack/` | Thin interactive entry point (`main()` → `app.Main`); holds the link-time `Build*` vars (`-ldflags -X main.Build...`). |
| `cmd/classicstack-svc/` | Windows service wrapper (SCM via `golang.org/x/sys/windows/svc`); `install`/`uninstall`/`start`/`stop`/`status`/`run`. Stub on non-Windows. |
| `cmd/classicstackd/` | Unix/macOS background daemon (self-daemonize via fork+`Setsid`, PID file); `start`/`stop`/`status`/`run`, plus macOS LaunchAgent `install`/`uninstall`. Stub on Windows. |
| `appletalk/` | DDP datagram struct, encode/decode, MacRoman codec |
| `router/` | Core routing engine, routing table aging, zone info |
| `port/ethertalk/` | EtherTalk over raw Ethernet using libpcap/Npcap, includes AARP |
| `port/localtalk/` | LocalTalk base; subpackages: LToUDP (UDP multicast 239.192.76.84:1954), TashTalk (serial at 1 Mbit/s), Virtual |
| `service/rtmp/` | Routing Table Maintenance Protocol — `RespondingService` + `SendingService` |
| `service/zip/` | Zone Information Protocol — `RespondingService` + `SendingService` |
| `service/afp/` | Apple Filing Protocol file server (largest subsystem, 35 files) |
| `service/asp/` | AppleTalk Session Protocol — AFP transport over DDP |
| `service/atp/` | AppleTalk Transaction Protocol — reliable messaging |
| `service/dsi/` | Data Stream Interface — AFP transport over TCP |
| `service/macip/` | IP-over-AppleTalk gateway with NAT and DHCP relay |
| `service/webui/` | Management web UI (`-tags webui`): HTTPS adapter over `pkg/control` — JSON API, SSE stats stream, embedded SPA |
| `pkg/control/` | Transport-agnostic management API (status, config stage/apply/save, service start/stop/restart, diagnostics); the single contract every UI front-end shares |
| `pkg/status/` | In-process service-status registry read by the dashboard |
| `pkg/metrics/` | Streaming stats hub (expvar + SSE sinks) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ObsoleteMadness/ClassicStack](https://github.com/ObsoleteMadness/ClassicStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
