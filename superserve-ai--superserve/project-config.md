---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

Superserve provides sandbox infrastructure to run code in isolated cloud environments powered by Firecracker MicroVMs. Users create sandboxes via the console, CLI, or SDK, execute commands, upload/download files, and manage sandbox lifecycle (pause/resume/delete).

This repo is a monorepo containing the console, CLI, TypeScript SDK, Python SDK, and UI library.

## Monorepo Structure

```
superserve/
├── apps/
│   ├── console/                 # Next.js 16 console app (App Router)
│   └── ui-docs/                 # UI component documentation (Vite, port 3003)
├── packages/
│   ├── cli/                     # TypeScript CLI (@superserve/cli on npm)
│   ├── python-sdk/              # Python SDK (superserve on PyPI) — hand-crafted
│   ├── sdk/                     # TypeScript SDK (@superserve/sdk on npm) — hand-crafted
│   ├── ui/                      # Shared UI component library (@superserve/ui)
│   ├── supabase/                # Supabase client factories (browser/server/admin/middleware)
│   ├── typescript-config/       # Shared tsconfig presets
│   └── tailwind-config/         # Shared Tailwind CSS config
├── tests/
│   ├── sdk-e2e-ts/              # TypeScript SDK end-to-end tests (Vitest)
│   └── sdk-e2e-py/              # Python SDK end-to-end tests (pytest)
├── docs/                        # Mintlify docs site (docs.json, MDX pages)
├── spec/                        # Planning and implementation documents
└── examples/                    # Example projects
```

**Workspace tooling:**
- **Bun workspaces** for dependency management (single `bun.lock` at root; workspaces: `apps/*`, `packages/*`, `tests/*`)
- **Turborepo** for task orchestration (build, lint, typecheck, test, e2e)
- **uv workspaces** for Python packages (`pyproject.toml` at root as workspace root)

## Architecture

### Console (`apps/console/`)

Next.js 16 App Router application with Supabase auth. Key architectural patterns:

**API Proxy** (`src/app/api/[...path]/route.ts`): All sandbox API calls from the browser go through a Next.js API route that authenticates the user via Supabase session, generates a server-side `X-API-Key`, and forwards the request to the platform API (`SANDBOX_API_URL`). The proxy key is cached for 24 hours per user.

**Server Actions** (`src/lib/api/*-actions.ts`): API keys, snapshots, activity, and audit logs are fetched directly via Supabase admin client in server actions, bypassing the proxy.

**Data Fetching**: React Query (TanStack Query) with custom hooks (`src/hooks/`). Mutations use optimistic updates. Query keys are centralized in `src/lib/api/query-keys.ts`.

**Auth**: Supabase Auth with Google OAuth and email/password. The proxy file (`src/proxy.ts`) handles route protection. Internal proxy API keys use the `__console_proxy__` name and are hidden from the UI.

**Analytics**: PostHog for event tracking. Events are defined in `src/lib/posthog/events.ts`.

### UI Library (`packages/ui/`)

Built on **@base-ui/react** (headless components) + Tailwind CSS + **motion** (Framer Motion) for animations. Component animations use CSS transitions with `data-starting-style`/`data-ending-style` attributes (defined in `packages/ui/src/styles/globals.css`). Icons from `@phosphor-icons/react`. Code highlighting via `shiki`.

### TypeScript CLI (`packages/cli/`)

Built with Bun + Commander. Entry point: `src/index.ts`. Authenticates via device flow or API key.

### TypeScript SDK (`packages/sdk/`) — v0.6.0

Published as `@superserve/sdk`. Hand-crafted SDK. Zero runtime dependencies (uses native `fetch`).

**Main API:**
- `Sandbox.create({ name })` / `Sandbox.connect(id)` / `Sandbox.list()` / `Sandbox.killById(id)`
- Instance: `sandbox.pause()` / `resume()` / `kill()` / `update()` / `getInfo()`
- Sub-modules: `sandbox.commands.run(cmd, opts)`, `sandbox.files.write/read/readText(path, ...)`
- Call `sandbox.kill()` or `Sandbox.killById(id)` to delete a sandbox — no `Symbol.asyncDispose` / `await using`

**Design choices:**
- `status` / `metadata` are `readonly` snapshots from construction — call `getInfo()` for fresh data
- `access_token` is private to the `sandbox` (not exposed in `SandboxInfo`); rotated automatically on `resume()` and re-injected into `sandbox.files`
- `kill()` is idempotent (swallows 404)
- `pause()` / `resume()` / `kill()` return `void` (no body); `getInfo()` / `list()` / `get()` return `SandboxInfo`
- Every network op accepts `AbortSignal`; GET/DELETE requests auto-retry transient errors (429, 5xx, network) with exponential backoff + jitter
- Streaming `run()` uses idle timeout (resets on each SSE chunk); throws if stream ends without a `finished` event
- `toSandboxInfo` throws on missing `id` / `status`; `create()` / `connect()` / `resume()` throw on missing `access_token`
- Typed errors: `SandboxError`, `AuthenticationError`, `ValidationError`, `NotFoundError`, `ConflictError`, `TimeoutError`, `ServerError`

### Python SDK (`packages/python-sdk/`) — v0.6.0

Published as `superserve` on PyPI. Hand-crafted SDK. Runtime deps: `httpx>=0.24.0`, `pydantic>=2.0.0`, `typing-extensions>=4.0.0`. Supports Python ≥ 3.9.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superserve-ai/superserve](https://github.com/superserve-ai/superserve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
