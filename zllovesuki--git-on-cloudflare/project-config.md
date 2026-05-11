---
trigger: always_on
description: This repository is a Git Smart HTTP v2 server implemented on Cloudflare Workers with Durable Objects, R2, KV, React SSR, and a small amount of client-side hydration.
---

# AGENTS.md

## Purpose

This repository is a Git Smart HTTP v2 server implemented on Cloudflare Workers with Durable Objects, R2, KV, React SSR, and a small amount of client-side hydration.

Write changes against the current source tree, not the docs alone. Some documentation is slightly behind the live layout.

## Rules from the user

- Reuse existing types/helpers/methods. Do not invent new types/helpers/method unnecessarily, especially do not use any or casting.
- Prefer clarity over cleverness. Favor explicit names and intermediate types over dense inline expressions. Small duplication is acceptable when it materially improves readability and maintenance.
- Comment the code, especially around nuanced behaviors and footguns. This is also not an excuse to be lean on comments.
- Prioritize lookup on `cloudflare-docs` mcp for up-to-date Cloudflare developer docs (if available). Fallback to searching the web if you cannot find the relevant information on this mcp

### Avoid transitive boundary crossings

- Try to keep cross-runtime boundaries to a single hop from the caller. Workers -> DO, Workers -> R2, or DO -> R2 are allowed, but do not chain boundaries transitively. In particular, Workers -> DO -> R2 is not allowed in most circumstances.
- Retain strict mutation boundaries between R2, Workers, and Durable Object: Durable Objects can hold a transaction against its object. Do not attempt to propose a design that resembles a distributed transaction (e.g. Workers reads a row, decide on resolutions, then invoke Durable Object RPC to mutate)
- If there needs to be resolution, Durable Object can resolve that conflict in a transaction and return a tagged union for Workers to decide
- If you need to reach for `blockConcurrencyWhile` in an Durable Object RPC, your design is probably wrong. Try again.
- In general: Workers stays stateless, no mutations; Durable Objects are stateful, transactional.

### Error handling

- Durable Objects do not throw. Use tagged union between Workers and Durable Objects to communicate outcome.
- Reserve `throw` in Durable Object for FUBAR

### No ad-hoc or duplicated types

- Before introducing a type, check whether a canonical one already exists. Key hubs:
  `src/git/core/objects.ts` (GitObjectType), `src/common/hex.ts` (OID helpers, zeroOid),
  `src/git/object-store/support.ts` (typeCodeToObjectType), `src/git/operations/limits.ts` (Limiter).
- Do not use `ReturnType<>`, `Awaited<ReturnType<>>`, or inline union literals when a named
  type already covers the shape. Extract a named type alias if one doesn't exist yet.
- Do not duplicate helpers. If you need `isZeroOid` or `typeCodeToObjectType`, import the
  existing one—don't redefine it locally.
- Watch for unused imports left behind after refactors; remove them.

### Keep comments in sync with code

- When modifying behavior, update every comment, JSDoc, and inline note that describes the
  old behavior in the same change. Outdated comments are worse than no comments.
- Review neighboring comments when editing a function—if the surrounding prose no longer
  matches the logic, fix it.

### Remove implementation-phase language

- Do not leave wording like "Phase 1/2/3", "TODO: next phase", "streaming-push WIP", or
  similar milestone markers in code, comments, or filenames once the feature is merged or
  the phase boundary is no longer meaningful. These create confusion for future readers.
- If a prior change left such references and you're editing the same area, clean them up.

### Visibility logging on new code paths

- Every non-trivial code path that touches R2, DO RPC, or background work must include
  structured logging using `createLogger` from `src/common/logger.ts` (or the DO's
  `this.logger`).
- Follow the established conventions: kebab-case message identifiers scoped by component
  (e.g. `"receive:finalize-committed"`), appropriate log level (debug for flow, info for
  state changes, warn for recoverable errors, error for hard failures), and structured
  extra fields with relevant context (oid, packKey, counts, etc.).
- When adding a branch or error path to an existing function that already logs, add
  matching visibility for the new path—don't leave silent gaps.

### Limiter usage on platform-bound calls

- Every R2 read/write and outbound DO RPC in a request-scoped code path must go through
  the `Limiter` from `src/git/operations/limits.ts` via `limiter.run(label, fn)`.
  Obtain the limiter with `getLimiter(cacheCtx)` or pass it through options.
- Use a descriptive label prefixed by target (e.g. `"r2:get-pack"`, `"do:get-object-compat"`).
- Respect the subrequest budget (`DEFAULT_SUBREQUEST_BUDGET = 900`). If the code path has
  its own budget (like `RECEIVE_SUBREQUEST_BUDGET`), use `countSubrequest()` to track it.
- Never bypass the limiter for "just one call"—the hard 1000-subrequest and
  6-concurrent-connection ceilings apply to the entire request, not individual call sites.

## Stack At A Glance

- Runtime: Cloudflare Workers with `nodejs_compat`
- Language: TypeScript ESM, strict mode
- UI: React 19 SSR via `react-dom/server`, client islands, Tailwind CSS v4, Vite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zllovesuki/git-on-cloudflare](https://github.com/zllovesuki/git-on-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
