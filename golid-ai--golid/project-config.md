---
trigger: always_on
description: Core codebase standards — applies to every conversation.
---


# Codebase Standards

## Backend (Go)

- **Parameterized queries only** — never `fmt.Sprintf` with values into SQL. Always use `$N` placeholders.
- **Transactions for multi-write only** — `pool.Begin` / `defer tx.Rollback` / `tx.Commit` for any operation with 2+ writes. Never wrap a single UPDATE/INSERT in a transaction — it adds overhead with no benefit.
- **Never discard errors** — no `_ = fn()`. Log with `logger.Error` at minimum. Fire-and-forget goroutines must wrap with error logging.
- **Check real DB errors before ErrNoRows** — `if err != nil && err != pgx.ErrNoRows { return 500 }`.
- **Resource membership, not just role** — verify the specific user belongs to the specific resource (item, record, etc.), not just that they're the right `userType`.
- **Parent status guards** — check parent entity status before mutations (e.g., reject updates on non-active resources).
- **`apperror` for all errors** — never `echo.NewHTTPError`. Use `apperror.BadRequest`, `.Forbidden`, `.NotFound`, `.Validation`, `.Internal`.
- **Config over hardcoded values** — platform fee percentages, pagination defaults, timeout durations, and retry counts go in `Config` struct or named constants, not inline in service methods. If you'd need to change it for a different environment, it belongs in config.
- **Request body size limit** — `middleware.BodyLimit("1M")` is configured in `stack.go`.
- **SSE uses one-time ticket auth (not JWT in URL)** — See `handler/sse.go` and `service/sse.go`.
- **Retry on fire-and-forget** — goroutines calling external services use `service.Retry()` with exponential backoff.
- **Shutdown order** — `e.Shutdown()` first (drain in-flight HTTP), then `sseHub.Shutdown()` (close SSE connections), then `defer db.Close()` (pool cleanup runs last via defer LIFO). Never close the DB pool before draining HTTP/SSE connections.

## Frontend (SolidJS)

- **`Switch/Match` for content states, NEVER nested `<Show>`** — nested `<Show when={!loading()}>` → `<Show when={error()}>` chains create stacked reactive scopes that leak computations during route transitions. Use flat `Switch/Match` with one `Match` per state.
- **`batch()` async signal updates** — `batch(() => { setData(result); setLoading(false); })` after every `await`. Prevents intermediate states during route transitions. Never use `try/catch/finally` with signals — `finally { setLoading(false) }` runs unbatched after `catch { setError(...) }`, creating an intermediate state. Instead, put `setLoading(false)` inside both `try` and `catch`, wrapped in `batch()`.
- **`onMount` + `defer: true`** — use `onMount` for initial fetch, `createEffect(on(..., { defer: true }))` for reactive refetches. Never let `createEffect` fire synchronously during mount.
- **No reactive expressions inside `<Title>`** — pre-compute as `createMemo`, pass resolved string. Inline reactivity in `<Title>` leaks during route transitions.
- **`onMount` + signals for all page data — no `createResource`** — the project uses `onMount` + `createSignal` + `alive` guard + `batch` for all data fetching. `createResource` causes orphaned computation warnings on route transitions and Suspense issues in conditional components. (`createAsync` + `query` from `@solidjs/router` is the official Solid 2.0 direction but not yet adopted.)
- **`onCleanup` for cleanup, NEVER return from `onMount` or `createEffect`** — SolidJS silently ignores return values from both `onMount` and `createEffect`. The React pattern `onMount(() => { ...; return () => cleanup() })` compiles without error but the cleanup never runs. Always use `onCleanup(() => cleanup())` inside the body.
- **`alive` guard on async** — `let alive = true; onCleanup(() => { alive = false; });` then check before setting signals after await.
- **`PRIVATE_ROUTES` in constants.ts** — add every new protected route or SSR auth redirects won't work.
- **`redirectTo` for login redirects** — SSR middleware redirects unauthenticated users to `/login?redirectTo=<encodedPath>`. The login page reads `searchParams.redirectTo` after auth succeeds. The param name is `redirectTo` — not `return`, `next`, or `redirect`.
- **`DestructiveModal` for destructive actions** — never `window.confirm()`.
- **Signal-driven modals** — `const [active, setActive] = createSignal(null)` for detail views within list pages, not sub-routes.
- **`lazy()` requires `<Suspense>`** — SolidJS `lazy()` components silently render nothing without a `<Suspense>` boundary. Always wrap `lazy()` usage in `<Suspense>`. SolidStart's router provides Suspense for route-level components, but `lazy()` inside a route needs its own `<Suspense>` wrapper.
- **`onCleanup` only synchronously** — never call `onCleanup()` inside an `async` function or after an `await`. SolidJS can't track ownership across async boundaries. Instead, declare mutable refs (e.g., `let observer: ResizeObserver | null = null`) and clean them up in the synchronous `onCleanup` registered during component creation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [golid-ai/golid](https://github.com/golid-ai/golid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
