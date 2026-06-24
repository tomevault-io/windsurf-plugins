---
trigger: always_on
description: manages the bearer itself and llama-dash passes it through.
---

# AGENTS

Notes for agents (and humans operating as agents) working on this repo.

## What this is

llama-dash is a dashboard UI plus a logging, auth-ready proxy for a local
inference backend. The implemented backend is currently
[llama-swap](https://github.com/mostlygeek/llama-swap), fronting llama.cpp
models through its `/v1/*` OpenAI/Anthropic-compatible endpoint. Feature ideas
and prioritization live in [`next-plan.md`](./next-plan.md).

Short version of goals:

- Single-pane dashboard for a self-hosted llama-swap stack.
- Add proxy features llama-swap doesn't have: API keys, quotas, filters, logging.
- Model lifecycle (load / unload / configure / hot-reload) driven from the UI.
- Keep inference backend integration capability-driven so future runtimes like
  Ollama can be added without weakening the llama-swap/GGUF path.
- Eventually shipped as a Docker Compose stack; llama-swap hidden internally.

Short version of non-goals:

- We do not run inference. That's the selected inference backend, currently
  llama-swap + llama.cpp.
- Not a multi-tenant SaaS. Single team, single box.
- No distributed deployment.

## Architecture

```
client ──► llama-dash :8080 ──► inference backend ──► model server procs
           │
           ├─ UI (/)
           ├─ Admin API (/api/*)
           └─ Proxy (/v1/*)  ← also what clients hit
```

One public port. The inference backend is not exposed on the host in the bundled
compose setup. The proxy layer is where auth, ACL, rate-limiting,
policies/filters, and logging all hang off.

## Repo layout

```
src/
  routes/                 — TanStack Router file-routes (UI + __root.tsx)
    index.tsx               · / dashboard home
    models.index.tsx        · /models list + load/unload actions
    models.$id.tsx          · /models/:id detail (stats, history, config snippet)
    requests.index.tsx      · /requests log with filtering, sorting, histogram
    requests.$id.tsx        · /requests/:id detail view
    keys.index.tsx          · /keys list + create/revoke/delete
    keys.$id.tsx            · /keys/:id detail (stats, model breakdown, requests)
    attribution.tsx         · /attribution header mapping + client setup examples
    policies.tsx            · /policies model aliases + request limits
    endpoints.tsx           · /endpoints client connection examples
    logs.tsx                · /logs raw log viewer
    login.tsx               · /login Better Auth username/password + passkey form
  features/               — route-owned UI, one component per file, grouped by feature
    auth/                  · login page
    dashboard/             · dashboard panels + metrics helpers
    endpoints/             · endpoint examples + copy/highlight helpers
    keys/                  · keys list/detail panels and forms
    models/                · models list/detail panels and helpers
    playground/            · playground tabs, rails, and media tools
    attribution/           · attribution settings page + setup examples
    policies/              · routing and request-limit editing panels
    requests/              · request list/detail pages and payload helpers
    config/, logs/         · config editor + log viewer feature-local pieces
  components/             — shared UI components reused across features
    Sidebar.tsx             · nav + VRAM-resident readout in footer
    ModelTimeline.tsx        · 30-min model swap timeline (spans + legend)
    Sparkline.tsx           · SVG sparkline with above-line glow
    DurationBar.tsx         · inline latency bar for request tables
    StatusDot.tsx           · animated status indicator (ok/warn/err/idle)
    StatusCell.tsx          · status code + stream badge
    PageHeader.tsx          · reusable kicker + title + subtitle + actions
    TopBar.tsx, Tooltip.tsx, ThemeToggle.tsx, CopyableCode.tsx
  lib/
    api.ts                — typed client-side fetch wrappers for /api/*
    queries.ts            — TanStack Query hooks (SSE refresh + slow polling fallback, infinite scroll)
    use-admin-events.ts   — EventSource bridge that invalidates query caches from /api/events
    schemas/              — valibot schemas (single source of truth for API types)
  server/                 — everything that runs in Node, never shipped to client
    auth.ts               — Better Auth dashboard session config + first-user signup guard + passkeys
    config.ts             — env-var loader (INFERENCE_BASE_URL, DATABASE_PATH, …)
    gpu-poller.ts         — polls nvidia-smi/rocm-smi/system_profiler for GPU stats
    model-watcher.ts      — polls /running every 15s, writes load/unload events
    db/                   — drizzle schema + SQLite init; migrations are applied explicitly with pnpm db:migrate
    proxy/                — /v1/* pass-through: context, handler, auth, body snapshots, transforms, forwarding, usage, queued logging, rate limits
    admin/                — /api/* admin surface: dispatcher plus grouped routes/, requests, model-events, model-detail, key-detail, api-keys, aliases, settings, events
    inference/            — selected inference backend facade plus backend-specific adapters and hints
    llama-swap/client.ts  — typed wrapper over llama-swap's HTTP API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndom91/llama-dash](https://github.com/ndom91/llama-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
