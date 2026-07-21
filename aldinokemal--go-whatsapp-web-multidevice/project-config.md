---
trigger: always_on
description: Generated: 2026-06-06
---

# PROJECT KNOWLEDGE BASE

Generated: 2026-06-06
Commit: 8c4ea8f
Branch: fix/chatwoot-postgres

## OVERVIEW

Go WhatsApp Web Multi-Device is a Go 1.25.5 WhatsApp Web API server with REST and MCP SSE modes.
It uses whatsmeow sessions, Fiber, plain Vue 3 modules, and SQLite-backed chat/session storage by default.

## STRUCTURE

```text
go-whatsapp-web-multidevice/
|-- src/                         # Go module root; run Go commands here
|   |-- main.go                  # go:embed views, then cmd.Execute
|   |-- cmd/                     # Cobra root, rest, mcp, global app wiring
|   |-- config/                  # Mutable package globals bound from flags/env
|   |-- domains/                 # Interfaces and DTOs; see child AGENTS
|   |-- usecase/                 # Business orchestration; see child AGENTS
|   |-- validations/             # ozzo-validation plus table tests
|   |-- ui/                      # REST, MCP, websocket adapters
|   |-- infrastructure/
|   |   |-- whatsapp/            # Device manager, events, presence pulse, JID utilities
|   |   |-- chatstorage/         # chat/message/device SQL repository
|   |   `-- chatwoot/            # Chatwoot REST sync and direct PG import
|   |       `-- pgimport/        # Direct Chatwoot Postgres importer; see child AGENTS
|   |-- views/                   # Embedded Vue 3 plain JS UI
|   |-- statics/                 # Runtime media, QR codes, send items
|   `-- storages/                # Runtime SQLite DBs and history dumps
|-- docs/                        # OpenAPI, webhook payload, Chatwoot docs
|-- docker/                      # Multi-stage Alpine image and entrypoint
|-- gallery/                     # Project logo assets
`-- .github/workflows/           # Docker publish, release, latest promotion
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add message type | `src/domains/send/`, `src/usecase/send.go`, `src/ui/rest/send.go` | REST is primary; MCP support is selective. |
| Add quoted reply support | `src/domains/send/`, `src/usecase/send.go`, `src/views/components/Send*.js` | Optional `reply_message_id`; use device-scoped quote lookup. |
| Add REST endpoint | `src/ui/rest/`, `src/usecase/`, `src/domains/` | Handler parses request, usecase validates/executes, domain owns DTO/interface. |
| Add MCP tool | `src/ui/mcp/` | Register in `Add*Tools`; resolve a device with `helpers.ContextWithDefaultDevice`. |
| Handle WhatsApp event | `src/infrastructure/whatsapp/event_*.go` | Register the concrete event in `event_handler.go`. |
| Presence behavior | `src/infrastructure/whatsapp/event_handler.go`, `presence_pulse.go`, `src/cmd/helpers.go` | Connect-time and scheduled pulse presence. |
| Add chat storage method | `src/domains/chatstorage/interfaces.go`, `sqlite_repository.go`, `chatstorage_wrapper.go` | Update domain, repository, and wrapper together. |
| Add DB migration | `src/infrastructure/chatstorage/sqlite_repository.go` `getMigrations()` | Append only. Current list has 29 migrations. |
| Add UI component | `src/views/components/`, `src/views/index.html` | Plain JS modules, no `.vue` single-file components. |
| Device management | `src/infrastructure/whatsapp/device_manager.go` | Central registry and purge/load/create logic. |
| Chatwoot integration | `src/infrastructure/chatwoot/` and `src/ui/rest/chatwoot.go` | REST sync, public webhook, optional direct Postgres import. |
| Direct Chatwoot import | `src/infrastructure/chatwoot/pgimport/` | Direct Chatwoot schema writes; see child AGENTS. |
| Chatwoot link/retry state | `src/infrastructure/chatstorage/sqlite_repository.go`, `src/infrastructure/whatsapp/webhook_forward.go` | Message links, read/delete sync, and persistent forward retries. |
| CLI flags / config | `src/cmd/root.go`, `src/config/settings.go`, `src/.env.example` | Flags and env mutate config package globals. |
| Shared helpers | `src/pkg/utils/`, `src/pkg/error/`, `src/pkg/sqlite/` | Utilities, aliased package errors, and CGO/purego SQLite driver selection. |
| Docker/release | `docker/golang.Dockerfile`, `.github/workflows/*.yaml` | Multi-arch Docker, tag/manual workflows, generated GoReleaser configs. |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `cmd.Execute` | function | `src/cmd/root.go` | Stores embedded views and runs Cobra root command. |
| `initApp` | function | `src/cmd/root.go` | Creates folders, DBs, WhatsApp client, device manager, repositories, and usecases. |
| `DeviceManager` | struct | `src/infrastructure/whatsapp/device_manager.go` | Owns active device registry and persisted device records. |
| `DeviceInstance` | struct | `src/infrastructure/whatsapp/device_instance.go` | Wraps per-device ID, JID, client, state, and storage. |
| `IChatStorageRepository` | interface | `src/domains/chatstorage/interfaces.go` | Storage contract for chats, messages, edits, calls, stats, schema, and device records. |
| `SQLiteRepository` | struct | `src/infrastructure/chatstorage/sqlite_repository.go` | Implements chat storage, Chatwoot link/retry state, and inline migrations. |
| `deviceChatStorage` | wrapper | `src/infrastructure/whatsapp/chatstorage_wrapper.go` | Injects or enforces device scoping for event-side storage access. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aldinokemal/go-whatsapp-web-multidevice](https://github.com/aldinokemal/go-whatsapp-web-multidevice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
