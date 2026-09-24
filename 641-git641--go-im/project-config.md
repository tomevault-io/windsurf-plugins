---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```bash
# Run the gateway server (reads configs/config.json)
go run ./cmd/gateway/

# Run the logic service (reads configs/config.json, requires MySQL)
go run ./cmd/logic/

# Run integration tests (spins up a real server on :18080)
go test ./cmd/gateway/ -v

# Run a single test
go test ./cmd/gateway/ -run TestIntegrationEndToEnd -v

# Build binaries
go build -o gateway.exe ./cmd/gateway/
go build -o logic.exe ./cmd/logic/
```

Unit tests (183 tests across 9 packages, all passing) are in `internal/pkg/snowflake/`, `internal/pkg/jwt/`, `internal/gateway/` (hub, router, redis_store, gnet_handler, grpc_client, hashring, grpc_gateway, group_store, unread_tracker, object_store, thumbnail, read_receipt, server), `internal/mq/` (producer, consumer), `internal/logic/` (gRPC server + consumer), `internal/repo/` (MySQL, skipped when not running), plus `cmd/gateway/` (5 integration tests: 2 WebSocket + 3 gnet TCP). Use `go test ./internal/...` to run them all, plus `go test ./cmd/gateway/ -v` for integration tests. See `docs/06-phase4-completion.md` for the full test breakdown.

## Architecture

This is a Go IM (instant messaging) system in MVP — a **single monolith** that merges three planned layers (Gateway → Logic → Storage). It uses Protocol Buffers (protobuf) binary encoding over WebSocket (gorilla/websocket) and raw TCP (panjf2000/gnet v2). See `docs/01-architecture-design.md` for the full vision.

### Request flow

```
# WebSocket (default, port :8080)
HTTP /login  → Server.HandleLogin              → JWT issued
HTTP /ws     → Server.HandleWS                 → JWT validated → WebSocket upgrade → Client created
WebSocket    → wsReadPump(ctx, conn, client, router) → Router.Route(ctx, …) → Hub
             → Client.WriteLoop()              → outbound messages (transport-agnostic)
             → wsPingLoop()                    → WebSocket ping/pong keepalive

# gnet TCP (optional, port :8081)
TCP connect  → GnetHandler.OnOpen              → pending state
TCP data     → GnetHandler.OnTraffic           → frame decode (4-byte len prefix + protobuf)
             → GnetHandler.processFrame        → handleLogin (first msg = CmdLogin + JWT)
                                               → Router.Route(ctx, …) via WorkerPool
             → Client.WriteLoop()              → outbound messages → gnet.Conn.AsyncWrite (4-byte len prefix + protobuf)

# Message history (CmdHistory)
CmdHistory    → Router.handleHistory(ctx, …)    → LogicClient.QueryHistory (gRPC) or MessageStore.QueryHistory (local)
             → sender.Send(historyMsg)        → sender.Send(completion) with Seq=delivered count

# Message persistence (Kafka, Phase 3)
CmdChat       → Router.handleChat(ctx, …)     → deliver + ACK (hot path, unchanged)
             → go mq.Producer.Publish(context.WithoutCancel(ctx), msg) → Kafka "im.message.persist" → mq.Consumer → MySQL

# Group chat (Phase 4)
CmdChat (ChatType=Group) → Router.handleChat → fanoutGroup(ctx, sender, msg)
                         → GroupStore.GetMembers → for each member: send or routeOrStoreOffline
                         → UnreadTracker.Increment (all members except sender)

# Read receipt & unread count (Phase 4)
CmdReadReceipt → Router.handleReadReceipt → UnreadTracker.MarkRead(reader, peer)
               → forward receipt to peer (local or cross-gateway via hash ring)
CmdUnreadCount → Router.handleUnreadCount → UnreadTracker.GetCounts(uid) → JSON response

# File upload/download (Phase 4)
HTTP /upload   → Server.HandleUpload      → JWT validate → ObjectStore.Put(fileID, data, mime)
                                          → Thumbnail generation (images, max 4096px, 200px output)
HTTP /file     → Server.HandleDownload    → JWT validate → ObjectStore.Get(fileID or fileID_thumb)

# Fulltext search (Phase 4)
CmdSearch      → Router.handleSearch     → MessageStore.SearchMessages(params) → paginated results + completion signal
HTTP /search   → Server.HandleSearch     → JWT validate → Router.Search → JSON response

# Message recall (Phase 5)
CmdRecall      → Router.handleRecall → validate target + sender + Seq (original MsgID)
               → msgStore.RecallMessage(msgID, fromUID) → MySQL marks recalled=1
               → recall notification to peer (local / cross-gateway / offline)
               → sendRecallError on failure — no ACK, no persistence for recall itself

# Multi-gateway forwarding (Phase 4)
routeOrStoreOffline → HashRing.Get(targetUID) → if peer owns: GrpcForwarder.Forward(ctx, uid, msg)
                    → GrpcGatewayServer.ForwardMessage → local deliver or StoreOffline
                    → fallback: local StoreOffline on forward failure
```

### Component responsibilities

| Component | File | Role |
|-----------|------|------|
| `App` | `cmd/gateway/main.go` | Wires everything together; owns `ClientRegistry`, `Server`, `Config`. Supports dual-transport (WebSocket + gnet TCP) startup via `transport` config. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [641-git641/GO-IM](https://github.com/641-git641/GO-IM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
