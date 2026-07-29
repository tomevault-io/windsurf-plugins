---
trigger: always_on
description: **Last Updated:** 2026-05-21
---

# Naming Conventions

**Status:** Active
**Last Updated:** 2026-05-21

This document is the canonical reference for naming patterns that recur across
the `notebooklm-py` codebase. It catalogues three families that an internal
architecture audit (findings CC2 / CC3 / CC5) called out as inconsistent enough
to need a written tiebreaker:

1. [Waiting / polling verbs](#1-waiting--polling-verbs-cc2) — `poll_X` vs
   `wait_for_X` vs `wait_until_X` vs `await_X` vs `_wait_for_X`.
2. [RPC-callable Protocol names](#2-rpc-callable-protocol-names-cc3) —
   `NextCall` / `RpcCallback` / `RpcCaller`.
3. [Metrics method verbs](#3-metrics-method-verbs-cc5) — `record_X` vs `emit_X`.

Examples below cite **symbol names only** (no file:line refs). Use
`rg '<symbol>' src/notebooklm/` to locate the current home — line numbers drift
faster than this file can keep up with.

---

## 1. Waiting / polling verbs (CC2)

Five distinct verbs are intentional. They are not interchangeable. Pick the
shape that matches what the function actually does and the loop will document
itself.

### `poll_X` — one-shot status read, no loop, no sleep

A `poll_X` function performs **a single** status / readiness check and returns
immediately. It never sleeps and never iterates. Use this when the *loop* lives
in the caller (or in a `wait_*` wrapper) and the function is just the
per-iteration probe.

Examples:

- `ArtifactPollingService.poll_status` — single RPC list + scan for one task ID.
- `ArtifactsAPI.poll_status` — public single-shot facade over the service.
- `ResearchAPI.poll` — single status read for a research plan.
- `artifact_poll` (CLI command) — one shot, then exit. Use the separate
  `artifact wait` command for the blocking / looping variant; `artifact poll`
  itself has no `--wait` flag.

> **Test name:** "if I call this twice in a row without a sleep, does that make
> sense?" If yes → it's a `poll_X`.

### `wait_for_X` — bounded loop with a **timeout**

A `wait_for_X` function loops until either the awaited condition holds **or** a
deadline expires. Timeouts are required (default or explicit); the function
raises a typed `*TimeoutError` on expiry rather than returning a sentinel.

Examples:

- `ArtifactPollingService.wait_for_completion` — loops `poll_status` until the
  artifact is terminal or `timeout` elapses.
- `ArtifactsAPI.wait_for_completion` — public facade over the service loop.
- `ResearchAPI.wait_for_completion` — loops `poll` until research is terminal,
  pinning a discovered `task_id` between iterations.
- `SourcePoller.wait_for_sources` (and `SourcesAPI.wait_for_sources`) — batch
  wait across N source IDs with a shared deadline.
- `RetryMiddleware._wait_for_rate_limit` / `_wait_for_server_error` — private
  variant; see the underscore-prefix subsection below.

### `wait_until_X` — loop on a **predicate** (also bounded)

`wait_until_X` reads like English: "wait until `X` is true". Same loop+timeout
contract as `wait_for_X`, but the verb signals that the awaited condition is a
**state predicate** on a specific resource, not the *arrival* of a value.

Examples:

- `SourcePoller.wait_until_ready` / `SourcesAPI.wait_until_ready` — block until
  `source.is_ready`.
- `SourcePoller.wait_until_registered` / `SourcesAPI.wait_until_registered` —
  block until a freshly-added source appears in the notebook listing.

> **`wait_for_X` vs `wait_until_X`:** both loop with a timeout. The difference
> is naming ergonomics. Prefer `wait_until_X` when the awaited condition is a
> boolean property of an existing resource (`is_ready`, `is_registered`).
> Prefer `wait_for_X` when you're waiting on an external arrival or a *set* of
> items (`wait_for_sources`, `wait_for_completion`). Neither form is "more
> correct"; pick the one that reads naturally at the call site.

### `await_X` — coalesced single-flight join

`await_X` is reserved for **single-flight coalescing** primitives: many
concurrent callers join one shared in-flight operation. The function name
matches the user-facing verb ("await the refresh"), and the implementation
guarantees deduplication (typically via `asyncio.shield` + a stored task).

Examples:

- `AuthRefreshCoordinator.await_refresh` — thundering-herd-safe token refresh;
  all 401-bouncing callers join one refresh task.

Do **not** use `await_X` for ordinary `async def` functions just because they
get `await`-ed. The verb signals coalescing semantics, not async-ness.

### `_wait_for_X` — module-private backoff helper

The leading underscore + `wait_for_` shape is used inside middlewares to
indicate **"this is the bounded backoff helper I extracted from one specific
retry leg"**. It is not a public coordination primitive; it is a private
implementation detail of a larger retry loop.

Examples:

- `RetryMiddleware._wait_for_rate_limit` — honors `Retry-After`, falls back to
  exponential backoff. Called from inside the rate-limit branch of the retry
  loop; never called externally.
- `RetryMiddleware._wait_for_server_error` — same shape for the 5xx branch.

If you extract a backoff helper from a middleware, follow this pattern. If you
extract a *public* waiting primitive, drop the underscore and use one of the
four verbs above.

### Summary table

| Verb | Loop? | Timeout? | Predicate or arrival? | Shared single-flight? | Public? |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teng-lin/notebooklm-py](https://github.com/teng-lin/notebooklm-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
