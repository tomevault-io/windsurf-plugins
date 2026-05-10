---
trigger: always_on
description: - When making examples (in docs, research, discussions, commit messages), use real-world business domain examples (e.g. supply chain disruption, customer refund, compliance audit), not developer-centric examples (e.g. merge PR, deploy service, fix bug). Osabio is a general-purpose coordination system, not a developer tool.
---

## Communication

- When making examples (in docs, research, discussions, commit messages), use real-world business domain examples (e.g. supply chain disruption, customer refund, compliance audit), not developer-centric examples (e.g. merge PR, deploy service, fix bug). Osabio is a general-purpose coordination system, not a developer tool.

## Git Commits

- Always use `--no-verify` when committing. The pre-commit hook requires `osabio init` which is not available in worktree environments.
- Always use `-s` (GPG sign) when committing.

## Deferred Work

- Deferred work (out-of-scope items, future enhancements, known limitations) must always be created as GitHub issues. Do NOT leave deferred work only as comments in code, TODOs in docs, or notes in wave-decisions files — create a GitHub issue so it is tracked and discoverable.

## Data Value Contract

- Never persist, publish, or return `null` for domain data values (Surreal records, API payloads, events, UI state).
- Absence must be represented by omitted optional fields only (`field?: Type`), not by `null`.
- If `null` appears in domain data, treat it as a contract violation and fix the producer. Do NOT sanitize/coerce it at consumers.

## TypeScript Conventions

- Do NOT use `null`. Use `undefined` via optional properties (`field?: Type`) instead.
- Do NOT create wrapper/helper functions for simple operations. Cast directly with `as`.
- Type result payloads once and avoid repetitive per-field casting.
- Do NOT use module-level mutable singletons (e.g. `let cache` at file scope) for caching or shared state. Module-level state is shared across the entire process — when multiple server instances run concurrently (e.g. smoke tests with `--concurrent`), they silently corrupt each other. Pass shared state via dependency injection or use per-instance caches scoped to the owning object.

## Internationalization

- Use the `Intl` API for all locale-sensitive formatting: `Intl.RelativeTimeFormat` for relative time, `Intl.DateTimeFormat` for dates, `Intl.NumberFormat` for numbers.
- Do NOT hand-roll formatting logic (e.g. custom "5m ago" strings). The `Intl` API handles locale, pluralization, and grammar rules automatically.

## Browser-Facing Route Authentication

- Browser-facing routes (UI pages, client-side fetches) must resolve identity from the Better Auth session, NOT from `X-Osabio-Identity` headers. The header-based pattern is for MCP/CLI clients only.
- Use `deps.auth.api.getSession({ headers: request.headers })` to get the session, then resolve identity via `identity_person` edge: `SELECT VALUE in FROM identity_person WHERE out = $person LIMIT 1`.
- Return 401 if no session or identity is found.
- See `tool-registry/routes.ts:resolveIdentityFromSession()` or `policy/policy-route.ts:resolveIdentityFromSession()` for reference implementations.

## Agentic Design: No Hardcoded Modes

- Do NOT introduce hardcoded processing modes (e.g. `"deterministic" | "llm"`) when behavior should be workspace-configurable via data.
- This is an agentic system — capabilities are defined by workspace admins through definitions, not by code branches. If something can be expressed as a definition with configurable logic, it should be.
- Avoid dual-path dispatchers that route between "built-in" and "dynamic" implementations. One path, driven by data. See ADR-038 for the precedent.

## Failure Handling

- Do NOT add fallback logic that masks invalid state, malformed payloads, or contract violations.
- Fail fast: throw immediately when required data is missing or does not match the expected shape.
- Prefer explicit hard failures over silent degradation, synthetic defaults, or "best effort" recovery.
- Only introduce fallback behavior when explicitly requested, and document the reason in code comments.
- Never silently ignore errors (e.g. empty `.catch(() => {})`). Always surface them via logging or re-throw.

## Graph Node Types

- Read @README.md § "Key Concepts" for graph node types and § "Architecture" for the layered architecture diagram.

## Server Architecture Overview

- Entrypoint is `app/server.ts`; it only calls `startServer()` from `app/src/server/runtime/start-server.ts`.
- Runtime bootstrap is split into:
  - `runtime/config.ts` (env parsing/validation)
  - `runtime/dependencies.ts` (Surreal + model clients)
  - `runtime/start-server.ts` (route registration + Bun server startup)
- HTTP cross-cutting concerns live in `app/src/server/http`:
  - `instrumentation.ts` (`withTracing()` — wide-event span per request with business context)
  - `response.ts` (JSON/headers helpers)
  - `parsing.ts` (request/form-data parsing)
  - `errors.ts` + `observability.ts` (error/log primitives)
- SSE state management is isolated in `app/src/server/streaming/sse-registry.ts`.
- Route/business domains are separated by workflow:
  - `auth/*` for authentication (Better Auth, OAuth, DPoP)
  - `workspace/*` for workspace create/bootstrap/scope checks
  - `chat/*` for ingress, chat agent, async message processing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osabiohq/osabio](https://github.com/osabiohq/osabio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
