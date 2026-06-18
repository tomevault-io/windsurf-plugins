---
trigger: always_on
description: Guide for AI agents working in this repository.
---

# AGENTS.md

Guide for AI agents working in this repository.

## Project Overview

ryu_ldn_nx is an Atmosphere (Switch CFW) sysmodule that bridges Nintendo's LDN (Local Data Network) multiplayer to Ryujinx's LDN servers over TCP. It ships as TitleID `4200000000000010` (boot2). Three IPC services register simultaneously:

1. **`ldn:u` MITM** — intercepts Nintendo's LDN user service. Implements `ICommunicationService` with a `CommState` state machine (None → Initialized → AccessPoint/Station → AccessPointCreated/StationConnected). Game-visible `NetworkInfo`, node IDs, and advertise data are synthesized from server responses.

2. **`bsd:u` MITM** — intercepts BSD socket calls. Sockets that `bind`/`connect` to the LDN subnet **10.114.x.x** are tracked as proxy sockets and their traffic is tunneled through `ProxyData` packets. This is how gameplay UDP/TCP traffic reaches Ryujinx servers without pcap.

3. **`ryu:cfg`** — custom non-MITM IPC service the Tesla overlay talks to for live configuration and status display.

All three services share a 384 KB expanded heap (`g_heap_memory`). The `new`/`delete` operators are overridden to route to this heap via `lmem::ExpHeap`. Do not grow buffers casually.

## Build Commands

All cross-compilation runs inside Docker (`devkitpro/devkita64`). Use docker compose, not bare `make` on the host.

```bash
# Sysmodule + overlay + dist ZIP (single command)
docker compose run --rm build

# Host unit tests (g++, not cross-compiled; uses -DTEST_BUILD)
docker compose run --rm test

# Clean all build artifacts and output/
docker compose run --rm clean
```

### Per-suite test targets

```bash
cd tests && make test-ldn-state-machine       # see Makefile for full list
make COVERAGE=1 coverage                        # gcov report
```

Available suite names: `protocol`, `config`, `config-manager`, `log`, `socket`, `tcp-client`, `connection-state`, `reconnect`, `client`, `ldn-types`, `ldn-state-machine`, `ldn-proxy`, `ldn-error`, `ldn-integration`, `overlay`, `ipc-config`, `config-ipc-service`, `shared-state`, `packet-dispatcher`, `session-handler`, `proxy-handler`, `handler-integration`, `upnp`, `p2p-proxy`, `p2p-client`, `p2p-integration`, `p2p-create-network`.

### Distribution packaging

```bash
docker compose run --rm build   # build + overlay + dist ZIP + output/ directory
```

The `dist` target in `sysmodule/Makefile` produces:
- `output/` directory with SD card structure (atmosphere/contents/..., switch/.overlays/..., config/)
- `ryu_ldn_nx-release.zip` with the same layout

Config file (`config.ini`) is NOT included in dist because `ensure_config_exists()` in `config.cpp` auto-creates it on first boot with defaults.

### Debugging

```bash
docker compose run --rm debugger <SWITCH_IP> [PID]   # interactive GDB session
```

GDB presets and component-specific debug scripts live in `scripts/debugger/`.

## Architecture & Control Flow

```
Game (ldn:u IPC)                Game (bsd:u IPC)              Tesla overlay
      │                               │                            │
      ▼                               ▼                            ▼
 ICommunicationService          BsdMitmService              ConfigService (ryu:cfg)
      │                               │                            │
      ▼                               ▼                            │
 LdnStateMachine ──▶ LdnPacketDispatcher                    LdnSharedState
      │                      │                                   ▲
      ▼                      ▼                                   │
 LdnSessionHandler    LdnProxyHandler ◀── P2pProxyClient/P2pProxyServer
      │                      │
      ▼                      ▼
 RyuLdnClient (network/)  ProxySocket / ProxySocketManager
      │                      │
      ▼                      ▼
 TcpClient + ConnectionState + Reconnect        BSD socket calls
      │
      ▼
 Ryujinx LDN server (TCP)
```

### Data flow for gameplay traffic (PIA mesh)

Games use Nintendo's PIA (Protocol Independent Application) library on top of LDN. After `CreateNetwork`/`Connect` establishes the LDN session via `ldn:u`:

1. Game calls `GetIpv4Address()` → gets virtual LDN IP (e.g., `10.114.0.1`)
2. Game calls `GetNetworkInfo()` → gets `NetworkInfo` with all nodes' IPs/MACs
3. Game identifies its own node by matching `GetIpv4Address()` result against `NetworkInfo.ldn.nodes[].ipv4Address`
4. Game opens UDP sockets via `bsd:u` → MITM intercepts `bind`/`connect` to `10.114.x.x` → creates `ProxySocket`
5. PIA sends broadcast UDP packets (mesh discovery) → `ProxySocketManager::RouteIncomingData()` delivers to **all** matching sockets (not just first)
6. PIA sends unicast UDP/TCP packets → `ProxySocket` → `ProxyData` header → TCP tunnel → Ryujinx server → peer

**Critical**: broadcast delivery must hit every listening socket on the same port. The `FindAllSocketsByDestination()` method replaces the old `FindSocketByDestination()` for broadcast routing to ensure PIA mesh discovery works.

### Packet signaling semantics

`HandleServerPacket` signals `m_response_event` **only** for actual response packets that `WaitForResponse` expects:
- **Signals event**: `Connected`, `ScanReply`, `ScanReplyEnd`, `RejectReply`, `ProxyConnectReply`, `NetworkError`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ethiquema/ryu_ldn_nx](https://github.com/Ethiquema/ryu_ldn_nx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
