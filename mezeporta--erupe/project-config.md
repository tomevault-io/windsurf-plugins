---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Erupe is a Go server emulator for Monster Hunter Frontier, a shut-down MMORPG. It handles authentication, world selection, and gameplay in a single binary running four TCP/HTTP servers. Go 1.25+ required.

## Build & Test Commands

```bash
go build -o erupe-ce                    # Build server
go build -o protbot ./cmd/protbot/      # Build protocol bot
go test -race ./... -timeout=10m        # Run tests (race detection mandatory)
go test -v ./server/channelserver/...   # Test one package
go test -run TestHandleMsg ./server/channelserver/...  # Single test
go test -coverprofile=coverage.out ./... && go tool cover -func=coverage.out  # Coverage (CI requires ≥50%)
gofmt -w .                              # Format
golangci-lint run ./...                 # Lint (v2 standard preset, must pass CI)
```

Docker (from `docker/`):
```bash
docker compose up db pgadmin            # PostgreSQL + pgAdmin (port 5050)
docker compose up server                # Erupe (after DB is healthy)
```

## Architecture

### Four-Server Model (single binary, orchestrated from `main.go`)

```
Client ←[Blowfish TCP]→ Sign Server (53312)      → Authentication, sessions
                       → Entrance Server (53310)  → Server list, character select
                       → Channel Servers (54001+) → Gameplay, quests, multiplayer
                       → API Server (8080)        → REST API (/health, /version, V2 sign)
```

Each server is in its own package under `server/`. The channel server is by far the largest (~200 files).

### Channel Server Packet Flow

1. `network/crypt_conn.go` decrypts TCP stream (Blowfish)
2. `network/mhfpacket/` deserializes binary packet into typed struct (~453 packet types, one file each)
3. `handlers_table.go` dispatches via `buildHandlerTable()` (~200+ `PacketID → handlerFunc` entries)
4. Handler in appropriate `handlers_*.go` processes it (organized by game system)

Handler signature: `func(s *Session, p mhfpacket.MHFPacket)`

### Layered Architecture

```
handlers_*.go  →  svc_*.go (service layer)  →  repo_*.go (data access)
                  (where needed)                     ↓
                                              repo_interfaces.go (21 interfaces)
                                                    ↓
                                              repo_mocks_test.go (test doubles)
```

- **Handlers**: Parse packets, call services or repos, build responses. Must always send ACK (see Error Handling below). Simple CRUD operations call repos directly; multi-step or cross-repo logic goes through services.
- **Services**: Encapsulate business logic that spans multiple repos or requires orchestration beyond simple CRUD. Not a mandatory pass-through — handlers call repos directly for straightforward data access.
- **Repositories**: All SQL lives in `repo_*.go` files behind interfaces in `repo_interfaces.go`. The `Server` struct holds interface types, not concrete implementations. Handler code must never contain inline SQL.
- **Sign server** has its own repo pattern: 3 interfaces in `server/signserver/repo_interfaces.go`.

#### Services

| Service | File | Methods | Purpose |
|---------|------|---------|---------|
| `GuildService` | `svc_guild.go` | 6 | Member operations, disband, resign, leave, scout — triggers cross-repo mail |
| `MailService` | `svc_mail.go` | 4 | Send/broadcast mail with message type routing |
| `GachaService` | `svc_gacha.go` | 6 | Gacha rolls (normal/stepup/box), point transactions, reward resolution |
| `AchievementService` | `svc_achievement.go` | 2 | Achievement fetch with score computation, increment |
| `TowerService` | `svc_tower.go` | 3 | Tower gem management, tenrourai progress capping, guild RP donation |
| `FestaService` | `svc_festa.go` | 2 | Event lifecycle (expiry/cleanup/creation), soul submission filtering |

Each service takes repo interfaces + `*zap.Logger` in its constructor, making it testable with mocks. Tests live in `svc_*_test.go` files alongside the service.

### Key Subsystems

| File(s) | Purpose |
|---------|---------|
| `sys_session.go` | Per-connection state: character, stage, semaphores, send queue |
| `sys_stage.go` | `StageMap` (`sync.Map`-backed), multiplayer rooms/lobbies |
| `sys_channel_server.go` | Server lifecycle, Raviente shared state, world management |
| `sys_semaphore.go` | Distributed locks for events (Raviente siege, guild ops) |
| `channel_registry.go` | Cross-channel operations (worldcast, session lookup, mail) |
| `handlers_cast_binary.go` | Binary state relay between clients (position, animation) |
| `handlers_helpers.go` | `loadCharacterData`/`saveCharacterData` shared helpers |
| `guild_model.go` | Guild data structures |

### Binary Serialization

`common/byteframe.ByteFrame` — sequential big-endian reads/writes with sticky error pattern (`bf.Err()`). Used for all packet parsing, response building, and save data manipulation. Use `encoding/binary` only for random-access reads at computed offsets on existing `[]byte` slices.

### Database

PostgreSQL with embedded auto-migrating schema in `server/migrations/`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mezeporta/Erupe](https://github.com/Mezeporta/Erupe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
