---
trigger: always_on
description: handles ADB dispatch via `SERVICE_ID_MAP`. See the
---

# OpenCompany - Claude Documentation

## Project Overview
This is a React Flow-based workflow automation platform implementing n8n-inspired architectural patterns. The project has undergone a comprehensive refactoring to implement modern INodeProperties interface system with full TypeScript compliance and code cleanup.

## Documentation Reference

**Always refer to these documentation files for detailed guides:**

| Document | Description |
|----------|-------------|
| **[Frontend Architecture](./docs-internal/frontend_architecture.md)** | Current frontend stack (React 19 + Vite + Tailwind v4 + shadcn/ui + Radix + RHF/zod + TanStack Query + Zustand). Tokens, primitives, forms, credentials exemplar, ownership boundary, `uiHints` catalogue. |
| **[UI Migration Plan](./docs-internal/ui_migration_plan.md)** | antd → shadcn/ui migration plan + completion log. Waves 1–10 done. Full frontend is schema-driven (backend SSOT); remaining DIY widget registry (ex-Phase 6) is the one deferred item. |
| **[Temporal Cleanup & Resilience Plan (Waves 15-18)](./docs-internal/TEMPORAL_CLEANUP_AND_RESILIENCE_PLAN.md)** | RFC for the four forward waves: 15 dead-code retirement (Redis-Streams branch in event_waiter, APScheduler cron stack, orphaned `connection_status` factory — ~556 LOC), 16 per-queue task routing activation (`TemporalWorkerPool` + `TEMPORAL_WORKER_POOL_ENABLED` gate), 17 resilience hardening (cron `catchup_window`, one-shot LLM-step retry, SDK interceptors, `DEPLOYMENT_MODE`, periodic heartbeats), 18 worker performance tuning (per-queue rate limits, sticky cache, poller autoscaling, resource-based slot supplier). Ship order, rollback channels, verified line citations. |
| **[Node Allowlist](./docs-internal/node_allowlist.md)** | Single-config UI visibility — `server/config/node_allowlist.json` controls which nodes / credential categories / skill folders show in the UI. Five lists with two enforcement tiers (mode-gated allowlist + absolute blocklist). `useNodeAllowlist` hook exposes `isVisible` / `isBlocked` / `isAllowed` / `isCredentialCategoryDisabled` / `isSkillFolderDisabled`. Adding a new disabled domain = single JSON edit, no code change. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeenie-ai/OpenCompany](https://github.com/zeenie-ai/OpenCompany) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
