---
trigger: always_on
description: A TCP proxy for Meshtastic LoRa mesh radio nodes, written in Go. It connects to a single Meshtastic node over TCP, accepts multiple client connections (iOS/Android Meshtastic app, Python CLI, etc.), and multiplexes traffic between them. Includes a web dashboard, mDNS service advertisement, and Kubernetes deployment manifests.
---

# Meshtastic Proxy — Agent Instructions

## Project Overview

A TCP proxy for Meshtastic LoRa mesh radio nodes, written in Go. It connects to a single Meshtastic node over TCP, accepts multiple client connections (iOS/Android Meshtastic app, Python CLI, etc.), and multiplexes traffic between them. Includes a web dashboard, mDNS service advertisement, and Kubernetes deployment manifests.

## Architecture

```
Meshtastic Node (TCP :4403)
        │
   node.Connection          ← persistent TCP, auto-reconnect, config caching
        │
   proxy.Proxy              ← accept clients, broadcast FromRadio, relay ToRadio
   ├── proxy.Client[0]      ← per-client read/write loops + send channel
   ├── proxy.Client[1]
   └── ...
        │
   Other components:
   ├── discovery.Advertiser  ← mDNS (_meshtastic._tcp), multi-interface support
   ├── web.Server            ← HTTP dashboard + SSE + metrics API
   ├── telegram.Bot          ← LoRa → Telegram bridge (optional, via pub/sub)
   └── metrics.Metrics       ← counters, ring buffers, pub/sub for SSE
```

### Package Map

| Package | Path | Purpose |
|---|---|---|
| `main` | `cmd/meshtastic-proxy/` | Entry point, wiring, signal handling |
| `config` | `internal/config/` | TOML config loading & validation |
| `node` | `internal/node/` | Persistent TCP connection to Meshtastic node, config cache |
| `proxy` | `internal/proxy/` | Client connection hub, broadcast, cached config replay |
| `protocol` | `internal/protocol/` | Meshtastic binary frame encoding (magic bytes + length + protobuf) |
| `discovery` | `internal/discovery/` | mDNS advertisement via hashicorp/mdns, multi-interface |
| `metrics` | `internal/metrics/` | Runtime stats, message log, traffic time-series, SSE pub/sub |
| `web` | `internal/web/` | HTTP server, dashboard, SSE endpoint |
| `telegram` | `internal/telegram/` | LoRa → Telegram bridge (optional, one-way text message forwarding) |

### Key Dependencies

- `buf.build/gen/go/meshtastic/protobufs` — Meshtastic protobuf definitions (FromRadio, ToRadio, MeshPacket)
- `github.com/hashicorp/mdns` — mDNS server (one `*mdns.Server` per network interface)
- `github.com/BurntSushi/toml` — config parsing
- `google.golang.org/protobuf` — protobuf runtime

## Wire Protocol

Meshtastic TCP uses a 4-byte frame header: `[0x94] [0xC3] [len_hi] [len_lo]` followed by a protobuf payload (max 512 bytes). See `internal/protocol/frame.go`.

- Node → Proxy: `FromRadio` protobuf (config, packets, telemetry, etc.)
- Proxy → Node: `ToRadio` protobuf (packets, want_config_id, etc.)
- Proxy → Client: same `FromRadio` frames (broadcast + cached config replay)
- Client → Proxy: same `ToRadio` frames (forwarded to node)

## Important Design Decisions

### Cached Config Replay

The proxy caches the full node configuration (MyInfo, Config, ModuleConfig, Channel, NodeInfo, ConfigCompleteId — up to 172+ frames) when it first connects to the node. Config is delivered to clients **on demand**: the proxy waits for the client to send `want_config_id` (which all standard Meshtastic clients do after TCP connect), then replies from cache via `replayCachedConfig`.

**Client connection flow in `proxy.handleNewConnection`:**
```
client = NewClient(conn, ...)   ← onMessage callback intercepts want_config_id & disconnect
registerClient(client)
client.Run(ctx)                 ← starts read/write loops; client sends want_config_id
                                ← onMessage intercepts it → replayCachedConfig(client, nonce)
```

`replayCachedConfig` delivers frames through `client.Send()` (the channel-based write loop). It substitutes the `ConfigCompleteId` nonce with the client's nonce so the client accepts the config sequence. The send channel buffer (256) is large enough for 172 frames because the write loop is actively draining it.

#### iOS Two-Phase Config (Special Nonces)

The iOS Meshtastic app uses two sequential `want_config_id` requests with special nonces defined in firmware (`PhoneAPI.h`):

| Nonce | Constant | Requests |
|---|---|---|
| `69420` | `SPECIAL_NONCE_ONLY_CONFIG` | Config only (MyInfo, Metadata, Channels, Config, ModuleConfig, own NodeInfo) — skips other nodes' NodeInfo |
| `69421` | `SPECIAL_NONCE_ONLY_NODES` | NodeInfo DB only (all NodeInfo frames) — skips config |

`filterConfigCache()` in `proxy.go` classifies cached frames and returns only the appropriate subset. For any other nonce (e.g. Python CLI with a random nonce), all frames are returned unfiltered.

#### iOS CoreData Timing Workaround (ownNodeInfoDelay)

When replaying cached config for nonce `69420` (config-only phase), the proxy inserts a 50ms pause **after sending the connected node's own NodeInfo frame**. This is a workaround for a timing issue in the iOS Meshtastic app:

1. iOS creates `NodeInfoEntity` in a CoreData **background context** when it receives the NodeInfo frame
2. The SwiftUI views read from the **view context**, which merges changes asynchronously via `automaticallyMergesChangesFromParent`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jf3tt/meshtastic-proxy](https://github.com/jf3tt/meshtastic-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
