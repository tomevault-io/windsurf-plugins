---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

All-Chat is a **cloud-native microservices platform** for aggregating and displaying chat messages from **multiple live streaming platforms** (Twitch, YouTube, Kick, TikTok, Discord) on streaming overlays with support for 7TV, BTTV, and FFZ emotes.

**Core Concept**: Users can create multiple overlays, each configured with one or more chat sources. An overlay can combine messages from Twitch + YouTube + Kick + TikTok + Discord simultaneously, providing full flexibility for streamers who multistream.

**Architecture**: Standard Go Layout with microservices communicating via Redis Streams (raw messages) → Message Processor (normalization + enrichment) → Redis Pub/Sub (overlay-specific) → API Gateway WebSocket (client delivery).

**Platform Status**:
- ✅ Twitch (IRC + EventSub) | ✅ YouTube (HTTP polling with quota tracking + InnerTube polling) | ✅ Kick (Pusher WebSocket) | ✅ TikTok (Unofficial library) | ✅ Discord (channel relay)

---

## Quick Start

```bash
# Full development environment (all services)
make docker-up         # Start postgres, redis, all services
make test              # Run tests
make migrate           # Apply database migrations (use `make migrate-down` to roll back)

# Frontend-only development (minimal backend)
make frontend-dev      # Start postgres, redis, gateway, overlay-manager, message-processor
make frontend-seed     # Create test overlay and chat sources
make frontend-messages # Generate mock chat messages
cd frontend && npm run dev  # Start frontend

# Access services
# - API Gateway: http://localhost:8080
# - Frontend: http://localhost:3000
```

**First Time Setup**: See [GETTING_STARTED.md](./GETTING_STARTED.md) for complete onboarding guide.

**Frontend Development**: See [FRONTEND_QUICK_START.md](./FRONTEND_QUICK_START.md) for minimal backend setup (30 seconds).
  - **All-in-one**: `make frontend-quick` (starts services, seeds data, verifies setup)
  - **Full docs**: [FRONTEND_DEV_SETUP.md](./FRONTEND_DEV_SETUP.md)
  - **File index**: [FRONTEND_FILES_INDEX.md](./FRONTEND_FILES_INDEX.md)

---

## Navigation by Task

### Common Tasks → Quick Reference Guides

**Need to...**

| Task | Guide | Lines |
|------|-------|-------|
| Add support for a new platform | [QUICK-REF-ADD-PLATFORM.md](./docs/llm-guides/QUICK-REF-ADD-PLATFORM.md) | ~150 |
| Debug YouTube quota issues | [QUICK-REF-DEBUG-QUOTA.md](./docs/llm-guides/QUICK-REF-DEBUG-QUOTA.md) | ~200 |
| Add a new HTTP endpoint | [QUICK-REF-ADD-ENDPOINT.md](./docs/llm-guides/QUICK-REF-ADD-ENDPOINT.md) | ~100 |
| Perform security audit | [QUICK-REF-SECURITY-AUDIT.md](./docs/llm-guides/QUICK-REF-SECURITY-AUDIT.md) | ~150 |
| Scale services or infrastructure | [QUICK-REF-SCALING.md](./docs/llm-guides/QUICK-REF-SCALING.md) | ~150 |
| Create database migration | [QUICK-REF-DATABASE-MIGRATION.md](./docs/llm-guides/QUICK-REF-DATABASE-MIGRATION.md) | ~100 |
| Debug Kubernetes issues | [QUICK-REF-KUBERNETES-DEBUG.md](./docs/llm-guides/QUICK-REF-KUBERNETES-DEBUG.md) | ~150 |
| Inspect Redis Streams/Pub/Sub | [QUICK-REF-REDIS-OPERATIONS.md](./docs/llm-guides/QUICK-REF-REDIS-OPERATIONS.md) | ~100 |

### Troubleshooting

**Having issues?** Start with the decision tree:

→ [Troubleshooting Decision Tree](./docs/troubleshooting/decision-tree.md) - High-level triage for all common issues

**Detailed troubleshooting guides** (created as needed):
- [build-errors.md](./docs/troubleshooting/build-errors.md) - Go compilation, Docker build, startup failures
- [connection-errors.md](./docs/troubleshooting/connection-errors.md) - PostgreSQL, Redis connection issues
- [youtube-quota-exceeded.md](./docs/troubleshooting/youtube-quota-exceeded.md) - Quota state machine, recovery procedures
- [twitch-irc-issues.md](./docs/troubleshooting/twitch-irc-issues.md) - IRC connection, channel join, message parsing
- [websocket-disconnects.md](./docs/troubleshooting/websocket-disconnects.md) - API Gateway WebSocket issues

### Architecture & Design Decisions

**Understand the system**:
- [Architecture Overview](./docs/architecture/00-OVERVIEW.md) - High-level design, service map
- [Data Flow](./docs/architecture/01-DATA-FLOW.md) - Message processing pipeline
- [Deployment](./docs/architecture/02-DEPLOYMENT.md) - Kubernetes architecture
- [Scaling](./docs/architecture/03-SCALING.md) - Performance and scalability
- [Observability](./docs/architecture/04-OBSERVABILITY.md) - Metrics, logging, tracing
- [Security](./docs/architecture/05-SECURITY.md) - Security architecture

**Understand WHY decisions were made**:
- [ADR Index](./docs/adr/README.md) - Architecture Decision Records
  - ADR-0001: Standard Go Layout (not hexagonal)
  - ADR-0002: Redis Streams + Pub/Sub hybrid
  - ADR-0003: CloudNativePG for PostgreSQL
  - ADR-0004: No ports/adapters abstraction
  - ADR-0005: React + Next.js frontend
  - ADR-0006: YouTube quota reserve-confirm-rollback
  - ADR-0007: Leadership rebalancing for auto-scaling

### Service Documentation

Each service has a detailed README:
- [api-gateway](./services/api-gateway/README.md) - WebSocket server, HTTP routing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caesarakalaeii/all-chat](https://github.com/caesarakalaeii/all-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
