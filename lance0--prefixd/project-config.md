---
trigger: always_on
description: This document provides context for AI agents working on prefixd.
---

# AGENTS.md - AI Agent Context for prefixd

This document provides context for AI agents working on prefixd.

## Project Overview

**prefixd** is a BGP FlowSpec routing policy daemon for automated DDoS mitigation. It receives attack events from detectors, applies policy-driven playbooks, and announces FlowSpec rules via GoBGP to enforcement points (Juniper, Arista, Cisco, Nokia routers).

## Architecture

```
                    ┌─────────────┐
                    │   nginx:80  │  ← single entrypoint
                    └──────┬──────┘
                     ╱            ╲
          ┌─────────┘              └──────────┐
          ▼                                   ▼
   dashboard:3000                      prefixd:8080
   (Next.js App Router)              (Rust/axum API)
                                          │
                          ┌───────────────┼───────────────┐
                          ▼               ▼               ▼
                   PostgreSQL:5432   GoBGP:50051    Prometheus:9090
                   (state store)    (FlowSpec BGP)  (metrics)
```

```
Detector → HTTP API → Policy Engine → Guardrails → FlowSpec Manager → GoBGP → Routers
                                           ↑
                                   Reconciliation Loop
                                           ↓
                                     PostgreSQL (state)
```

## Directory Structure

```
src/
├── alerting/          # Webhook alerting: Slack, Discord, Teams, Telegram, PagerDuty, OpsGenie, generic
├── api/
│   ├── handlers.rs    # All HTTP handlers (health, events, mitigations, config, safelist, operators, alerting)
│   ├── routes.rs      # Route definitions: public_routes(), session_routes(), api_routes(), common_layers()
│   ├── openapi.rs     # utoipa OpenAPI spec registration
│   └── metrics.rs     # HTTP request metrics middleware
├── auth/              # AuthBackend (axum-login), mode-aware auth (none/bearer/credentials/mtls)
├── bgp/               # FlowSpecAnnouncer trait, GoBGP gRPC client, mock
├── config/            # Settings, Inventory, Playbooks (YAML parsing)
├── correlation/       # Multi-signal correlation engine (config, engine, signal groups, webhook adapter)
├── db/                # PostgreSQL repository with sqlx + MockRepository for testing
├── domain/            # Core types: AttackEvent, Mitigation, FlowSpecRule
├── guardrails/        # Validation, quotas, safelist protection
├── observability/     # Tracing, Prometheus metrics
├── policy/            # Policy engine, playbook evaluation
├── scheduler/         # Reconciliation loop, TTL expiry
├── ws/                # WebSocket handler and message types
├── error.rs           # PrefixdError enum with thiserror
├── state.rs           # Arc<AppState> with shutdown coordination, RwLock for inventory/playbooks/alerting
├── lib.rs             # Public module exports
├── main.rs            # CLI, daemon startup
└── bin/prefixdctl.rs  # CLI tool for controlling the daemon

frontend/
├── app/
│   ├── (dashboard)/           # Route group with RequireAuth + ErrorBoundary layout wrapper
│   │   ├── layout.tsx         # Auth guard + WebSocket + ErrorBoundary for all dashboard pages
│   │   ├── page.tsx           # Overview
│   │   ├── mitigations/       # Mitigations list with inline withdraw
│   │   ├── mitigations/[id]/  # Mitigation detail (full-page, timeline, customer context)
│   │   ├── events/            # Event log
│   │   ├── inventory/         # Searchable customer/service/IP browser
│   │   ├── audit-log/         # Audit trail
│   │   ├── config/            # Settings (JSON) + Playbooks (cards) + hot-reload
│   │   ├── admin/             # Tabbed: System Status, Safelist CRUD, User management
│   │   ├── ip-history/        # IP history timeline with search
│   │   └── correlation/       # Correlation dashboard (Signals, Groups, Config tabs) + group detail
│   ├── login/                 # Login page (outside auth guard)
│   ├── globals.css            # Light + dark theme variables
│   └── layout.tsx             # Root layout with ThemeProvider + Toaster
├── components/
│   ├── dashboard/             # Sidebar, top-bar, BGP status, command palette, detail panels
│   ├── ui/                    # shadcn/ui components (button, card, dialog, alert-dialog, etc.)
│   ├── error-boundary.tsx     # React class ErrorBoundary with retry button
│   ├── websocket-provider.tsx # Centralized WebSocket context (SWR invalidation + toasts)
│   ├── require-auth.tsx       # Auth-mode-aware guard with deny-by-default
│   └── swr-provider.tsx       # SWR config with 401 retry suppression
├── hooks/
│   ├── use-api.ts             # SWR hooks for all endpoints
│   ├── use-auth.tsx           # AuthProvider with session expiry listener
│   └── use-permissions.ts     # Role-based permissions (deny-by-default, settled flag)
├── lib/
│   ├── api.ts                 # Fetch wrapper, all API functions, 401 debounce dispatch
│   └── mock-api-data.ts       # Mock data for development
├── __tests__/                 # Vitest tests
├── vitest.config.ts           # Vitest config (jsdom, react plugin, @ alias)
└── vitest.setup.ts            # jest-dom matchers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lance0/prefixd](https://github.com/lance0/prefixd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
