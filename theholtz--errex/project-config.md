---
trigger: always_on
description: This file is binding for any agent or contributor working in this repo. It
---

# errex — engineering rules

This file is binding for any agent or contributor working in this repo. It
captures non-obvious project constraints. Read it before touching code.

## Project shape

- `crates/errex-proto` — wire types shared across the workspace (Issue, Event,
  Fingerprint, ServerMessage, ClientMessage). Pure data + serde.
- `crates/errexd` — error-tracking daemon. HTTP ingest (Sentry envelope
  compatible), WebSocket fan-out, SQLite persistence, embedded SvelteKit SPA
  via `rust-embed`. Single binary.
- `web/` — SvelteKit 5 SPA (TypeScript strict, Tailwind v4, Svelte runes).
  Static-adapter build is bundled into the daemon at compile time.
- `docker/` — multi-stage Dockerfile (bun → cargo → distroless), compose file.
- `scripts/dev.sh` — boots daemon + Vite dev server in parallel.
- `scripts/seed.sh` — seeds the daemon with a realistic mix of issues.
- `scripts/smoke.sh` — boots a release binary and probes the API.
- `errex.sh` — task runner; everything that touches Rust runs in
  `rust:1-slim` so a local toolchain is not required.

## Hard non-functional constraints

1. **Lightweight first.** errex targets self-host on small machines. RAM is a
   first-class constraint, not an afterthought.
   - Prefer DB queries to in-memory caches when SQLite is fast enough.
   - Bound every channel and buffer; never let a queue grow unbounded.
   - Keep the sqlx pool small (≤4 connections); the digest task is single-writer.
   - Don't preload payloads. Stream where possible.
   - If you must cache, justify it with a measured number, not "for speed".

2. **Self-host friendly.** Single binary, SQLite (no Postgres dep), zero
   required external services. SPA is embedded — no separate web server.

3. **Sentry-SDK compatible ingest.** `/api/:project/envelope/` accepts the
   real envelope format. SDKs should "just work" by pointing at errexd.

## Testing — TDD is mandatory (Rust AND frontend)

**Rule:** every code change ships with tests. New behavior gets a failing
test FIRST, then the implementation that makes it pass. Bug fixes get a test
that reproduces the bug, then the fix.

This applies to **Rust** (`crates/`) and the **frontend** (`web/`). The
specific test surfaces and runners differ per stack — see below.

### Rust (`crates/`)

What MUST be tested:

- Every public method on `Store` (`upsert_issue`, `insert_event`,
  `latest_event`, `load_issues`, `list_issues_by_project`,
  `project_summaries`, `set_status`, project/webhook helpers, etc.).
- The fingerprint algorithm (`crate::fingerprint::derive`) — stability
  across normalization edge cases. Fingerprint regression is a silent UX
  catastrophe (issues fragment), so this gets generous coverage.
- The Sentry envelope parser (`crate::ingest::parse_envelope`) — header,
  length-prefixed items, gzip detection, malformed inputs.
- Wire types in `errex-proto`: serialize ↔ deserialize round-trips of
  `Event`, `Issue`, `ServerMessage`, `ClientMessage`, `IssueStatus` so any
  breaking change to JSON shape fails CI loudly.
- HTTP routes via `axum::Router` `oneshot` requests against an in-memory
  `AppState` (status codes, body shapes, auth boundaries).

How to write tests:

- Integration tests live in `crates/<name>/tests/*.rs`. Each file is its
  own binary; `mod` the source files via `#[path]`.
- Pure unit tests live in `#[cfg(test)] mod tests` blocks at the bottom of
  the source file.
- Tempdir + fresh SQLite for any test that touches `Store`. Never reuse
  `./data/errex.db`.
- No mocks of `Store`; use the real type with a tempdir.
- Tokio tests use `#[tokio::test]`.

### Frontend (`web/`)

What MUST be tested:

- Every module under `web/src/lib/` that contains pure logic
  (`api.ts`, `eventDetail.ts`, `eventStream.svelte.ts`, `actions.svelte.ts`,
  `selection.ts`, `stores.svelte.ts`, `utils.ts`, `toast.svelte.ts`,
  `ws.ts`).
- New behavior in components when the behavior is NOT purely visual —
  filter logic, command palette dispatch, keyboard shortcut handling,
  state transitions in modals.

What does NOT need tests:

- Pure visual styling (Tailwind classes, layout decisions).
- Component rendering output verified against a snapshot (snapshot tests
  rot fast and don't catch real bugs at this size).

How to write tests:

- Vitest, jsdom env. Files are `*.test.ts` colocated with the module
  they cover, e.g. `api.test.ts` next to `api.ts`. The runner picks them
  up from anywhere under `src/`.
- For Svelte 5 stores (`*.svelte.ts`), test by importing the singleton and
  invoking its methods — runes work in test context.
- For HTTP code, mock `fetch` via `vi.spyOn(globalThis, 'fetch')`.
- Component tests use `@testing-library/svelte` only when needed (event
  dispatch, conditional rendering tied to logic). Visual regressions are
  not in scope.

### How to run

- Rust: `./errex.sh check` runs `fmt --check`, `clippy -D warnings`,
  `cargo test --workspace` inside the rust container.
- Frontend: `bun test` (or `bun run check && bun test`) inside `web/`.
- Combined gate: a green PR has both green. CI runs both.

### TDD workflow (required)

1. Write the test that describes the desired behavior. Run it; confirm it
   fails for the right reason (compile error, missing import, assertion
   mismatch — all valid "red").

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheHoltz/errex](https://github.com/TheHoltz/errex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
