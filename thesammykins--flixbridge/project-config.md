---
trigger: always_on
description: This document defines engineering principles, architectural constraints, and contribution patterns for this MCP server. It is intentionally focused on *how* we build (process, quality bar, patterns) rather than *what* we build. Keep it short, boring, and repeatable.
---

# AGENTS.md – How We Build Here

This document defines engineering principles, architectural constraints, and contribution patterns for this MCP server. It is intentionally focused on *how* we build (process, quality bar, patterns) rather than *what* we build. Keep it short, boring, and repeatable.

---

## 0. Core Constraints

1. Total handwritten runtime TypeScript LOC (excluding config, README, this file) MUST stay ≤ 700 lines, but the entire project can be more than that, any core service is exempt from this rule, but it should be kept as small as possible.
2. Zero heavy frameworks. Prefer the platform (Node 20+ LTS, native `fetch` / `undici`).
3. Minimal dependencies (target set):
   - `typescript@^5.9.x`
   - `zod` (runtime validation, narrow usage)
   - `@biomejs/biome` (formatter + linter; replaces ESLint/Prettier)
   - `tsx` (optional dev runner) OR plain `node` as appropriate
4. No class hierarchies unless polymorphism **cannot** be expressed more simply (object literals + functions preferred).
5. Strict TypeScript (`"strict": true`, include `noUncheckedIndexedAccess`, `useUnknownInCatchVariables`).
6. All external I/O (HTTP) funneled through one tiny helper to enforce:
   - Timeouts
   - Error normalization
   - Consistent logging hooks (single choke point)
7. The repository should always be in a shippable state (main = releasable).
8. Public surface area (exported symbols) stays tiny; internal refactors remain cheap.

---

## 1. High-Level Architecture

```
/src
  index.ts                 -> MCP server bootstrap + tool registration
  core.ts                  -> shared fetch wrapper (retry + timeout)
  debug.ts                 -> debug logging system (Phase 3)
  metrics.ts               -> observability & metrics collection (Phase 3)
  services/
    base.ts                -> shared service & operation type contracts
    shared.ts              -> BaseArrService implementation
    registry.ts            -> service registry: serviceName -> adapter
    arr/
      sonarr.ts            -> Sonarr adapter (conforms to shared contract)
      radarr.ts            -> Radarr adapter (conforms to shared contract)
    downloaders/
      sabnzbd.ts           -> SABnzbd downloader integration (Phase 2)
```

Rationale:
- `services/*` know raw REST shapes (endpoint paths, differences).
- `ops/*` expose normalized *intent-level* functions. They choose the right service adapter & endpoint.
- `index.ts` binds these operations as MCP tools (each tool intentionally coarse-grained).
- `http.ts` centralizes REST calling + error mapping.
- `mapping.ts` allows dynamic extension (future: add `lidarr`, etc.) without changing ops.

---

## 2. Service Abstraction Model

We unify common operations (see differences file) into a normalized capability contract.

```ts
// Pseudocode sketch (not final API)
export interface BaseService {
  id: 'sonarr' | 'radarr' | string;
  baseUrl: string;
  apiKey: string;
  // Raw endpoint path fragments (no query)
  paths: {
    systemStatus: string;             // "/api/v3/system/status"
    queue: {
      list: string;                   // "/api/v3/queue"
      details: string;                // "/api/v3/queue/details"
      status: string;                 // "/api/v3/queue/status"
      grab: (id?: number) => string;  // "/api/v3/queue/grab/{id}" or bulk
    };
    history: {
      base: string;                   // "/api/v3/history"
      since: string;                  // "/api/v3/history/since"
      detail: string;                 // "/api/v3/history/series" | "/api/v3/history/movie"
      failed: (id: number) => string; // POST path
    };
    mediacover: (mediaId: number, filename: string) => string;
    // Divergences captured as simple variant strings
    blocklist: {
      base: string;                   // "/api/v3/blocklist"
      scoped?: string;                // Radarr adds "/api/v3/blocklist/movie"
    };
    lookup: {
      base: string;                   // "/api/v3/series/lookup" or "/api/v3/movie/lookup"
      tmdb?: string;                  // Radarr only
      imdb?: string;                  // Radarr only
    };
  };
}
```

Key points:
- Divergent resource nouns become path variants (e.g. `series` vs `movie`).
- Optional properties indicate feature not present on all services.
- NO inline fetch logic inside adapters: they only produce URLs and supply semantic hints.
- Operation layer decides HTTP verb / body shape (shared semantics).

---

## 3. Operation Normalization

We define an “operation catalog” with stable semantic names. Example subset:

| Semantic Operation | Sonarr Path(s)                             | Radarr Path(s)                              | Notes |
|--------------------|--------------------------------------------|---------------------------------------------|-------|
| systemStatus       | /api/v3/system/status                      | /api/v3/system/status                       | Identical |
| queueList          | /api/v3/queue                              | /api/v3/queue                               | Identical |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thesammykins/FlixBridge](https://github.com/thesammykins/FlixBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
