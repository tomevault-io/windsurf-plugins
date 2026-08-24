---
trigger: always_on
description: **JSLab** is a multi-service platform for visualizing JavaScript engine internals (V8, SpiderMonkey, JavaScriptCore, Hermes). The architecture follows a microservices pattern with isolated engine services, a Fastify API gateway, and a Next.js frontend UI.
---

# JSLab Copilot Instructions

## Project Overview

**JSLab** is a multi-service platform for visualizing JavaScript engine internals (V8, SpiderMonkey, JavaScriptCore, Hermes). The architecture follows a microservices pattern with isolated engine services, a Fastify API gateway, and a Next.js frontend UI.

### Service Architecture

```
Client → Next.js Frontend → Fastify API Gateway (Redis cache + rate limit)
                              ↓
                    Engine HTTP Services
                    ├─ engine-v8 (d8; flags decide the output)
                    ├─ engine-hermes (hermes -dump-bytecode)
                    ├─ engine-jsc (jsc -d)
                    ├─ engine-spidermonkey (js, dis() wrapper)
                    └─ trace-service (engine262 in a worker thread)
```

**Key constraint**: Engine services are stateless HTTP wrappers around CLI tools. No shared state and no inter-service communication exists — only the API gateway orchestrates requests. The only filesystem they touch is a per-request temp dir under `/tmp` (an `emptyDir`, since the root filesystem is read-only), which holds the snippet and any shim scripts and is removed when the run ends.

## Critical Developer Workflows

### Local Development (Skaffold + k3s)

```bash
# Everything, no cluster needed (recommended)
docker compose up --build

# Full k8s stack with auto-rebuild
skaffold dev --port-forward -n jslab

# Individual app development
cd packages/engine-runtime && npm ci && npm run build  # api + engines need this first
cd apps/api && npm run dev          # Fastify @ localhost:8080
cd apps/frontend/src && npm run dev # Next.js @ localhost:3000
cd apps/engine-v8 && npm run dev    # Engine @ localhost:8080
```

The trace service vendors engine262 as a git submodule — clone with
`--recurse-submodules`, or run `git submodule update --init --recursive`.

### Build & Deploy

- **Docker images**: Each app has a `Dockerfile` with `dev` and `prod` targets (see `apps/*/Dockerfile`). The api and the four engines bake in `packages/engine-runtime`, so they build **from the repo root**: `docker build -f apps/api/Dockerfile -t jslab-api .`. The frontend and trace-service build from their own directory.
- **Kubernetes**: `kubectl apply -k infra/k8s/base` (namespace: `jslab`).
- **Ingress**: four Traefik Ingress objects share the hosts and are ordered by explicit router priorities — `/api/trace/functions` + `/api/spec` → frontend (2000), `/api` → api (1000), `/embed` → frontend with relaxed frame headers (500), `/` → frontend (10). See `infra/k8s/base/ingress.yaml` and `infra/README.md`.

### Type Safety & Validation

- All services use **Zod** for schema validation and type inference (e.g., `apps/api/src/schemas.ts`, `packages/engine-runtime/src/index.ts`).
- **TypeScript** with `"type": "module"` and ESM imports throughout.
- Run `npm run lint` (tsc --noEmit) in each app to check types.

## Data Flow & Contract Types

### API Request/Response Contract

See [apps/api/src/types.ts](../apps/api/src/types.ts):

```typescript
type RunRequest = {
  engine: "v8" | "hermes" | "sm" | "jsc";
  sourceText: string;
  options?: { flags?: string[]; timeoutMs?: number };
};

type ApiResponse = {
  ok: boolean;
  stdout: string;
  stderr: string;
  artifacts: Artifact[];
  meta: { durationMs: number; engine: string; cacheHit: boolean; ... };
};
```

**Critical points**:

- Frontend → API: POST `/api/run` with `RunRequest`.
- API → Engine services: POST `/run` (same schema).
- All engine services filter flags against the shared `flagCatalog` (e.g., V8: `--print-bytecode`, `--trace-ignition`; Hermes: `-O`, `-gc-sanitize-handles`). Rejected flags are reported back in `meta.droppedFlags`.
- `artifacts` is part of the contract but is currently always `[]` — every engine returns its output as text on stdout/stderr.
- `sourceText` is immutable; flags and timeout are normalized server-side (`timeoutMs` clamped to `[MIN_TIMEOUT_MS, MAX_TIMEOUT_MS]` = `[250, 5000]`, source capped at `MAX_SOURCE_LENGTH` = 20000).
- **Cache key** includes engine, sourceText, flags, and a timeout bucket (Math.ceil(timeoutMs/100)) to reduce cache misses on timeout variations.

## Essential Patterns

### 1. Engine Service Template

All engines ([engine-v8](../apps/engine-v8/src/server.ts), [engine-hermes](../apps/engine-hermes/src/server.ts), etc.) are thin wrappers around the shared [packages/engine-runtime](../packages/engine-runtime/src/) package:

- Each `server.ts` calls `startEngineServer()` with an `EngineSpec`: the engine name (also the flag-catalog key), a temp-dir prefix, its config, the globals to lock down, any prelude scripts, and an `invoke()` callback that builds the binary command line. For V8, flags drive all behavior (e.g. `--print-bytecode` to get bytecode output).
- Each `config.ts` extends `engineEnvBase` from the shared package with only its own fields (the binary path, v8's heap cap) and loads it with `loadEnv()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pavlof01/jslab](https://github.com/pavlof01/jslab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
