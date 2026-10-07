---
trigger: always_on
description: Last revised: 2026-09-10
---

# Suprnova Live -- Conventions

Status: Normative
Last revised: 2026-09-10

## Authority and application

These conventions govern Suprnova Live design, implementation, generated code,
browser assets, component-library artifacts, tests, references, and
documentation. The machine production coding standard and any future
repository-local `BEST_PRACTICES.md` remain authoritative. A repository-local
`BEST_PRACTICES.md` overrides the machine copy.

`crates/suprnova-live/` is the internal Suprnova Live engine subtree, not a
specification-only repository or a third-party crate. It keeps implementation
beside the normative `docs/specs/suprnova-live/` set and
`scripts/check-specs.mjs`; those files, browser sources and artifacts, fixtures,
tests, benchmarks, and implementation documents are one maintained authority.
The former `/home/shawn/workspace2/suprnova-live` checkout is immutable
historical provenance only. Its large `reference/` catalog and optional
`suprnova-live.zip` Fable handoff export remain non-normative historical
artifacts and do not become a parallel current contract. The active iteration
contract named under Completeness and scope authorizes coherent changes in an
isolated Suprnova worktree while unrelated Suprnova and Magnetar work remains
untouched.

## Implementation standards

### Completeness and scope

- The active implementation contract is [`iterations/006.md`](iterations/006.md).
  Closed contracts remain historical evidence; preliminary sequencing in an
  older contract does not override the active confirmed boundary.
- Implement the active iteration contract completely. Do not substitute an
  MVP, placeholder, TODO, empty adapter, unverified scaffold, or narrower
  behavior for an agreed capability.
- A change owns its whole contract: implementation, public facade, macro/checker
  metadata, browser behavior, tests, documentation, generated templates,
  reference impact, and migration note where applicable.
- Attractive adjacent work is not implicit scope. Record it through
  `/next-iteration` after Stage 6 rather than coupling it to the current change.
- Existing unrelated work in Suprnova, Magnetar, or this workspace is preserved.
  Never rewrite, revert, reformat, or commit another contributor's changes merely
  to simplify Live work.

### Rust safety and API design

- Public application-facing APIs live under `suprnova::live` or
  `suprnova::view`; consumers never import `suprnova-live` directly.
- The internal engine crate does not depend on the `suprnova` facade. Framework
  services enter through narrow adapter traits and typed contexts to prevent a
  crate cycle and make conformance fixtures independent.
- Public operations return typed `Result` values and preserve causal errors.
  Panics are limited to proven internal invariants in tests or unreachable
  generated states; hostile input, provider failure, and application mistakes
  are never panic paths.
- Production code contains no `unsafe`; the engine crate keeps
  `#![forbid(unsafe_code)]`. The `suprnova-live` package lint is `deny`
  rather than `forbid` for one recorded reason: `benches/render_cache_budget.rs`
  allows `unsafe` with a written reason for the counting global allocator that
  measures the Complete L0 allocation budget, and that benchmark holds the only
  `unsafe` in the subtree. A later proposal requiring `unsafe` needs a
  separately approved, documented safety case and cannot enter as an
  implementation convenience.
- Public items have useful rustdoc because `suprnova` denies missing docs and
  broken or private intra-doc links.
- Clippy warnings are reviewed and resolved, but commands and gates do not use
  blanket `-D warnings`. An intentional lint suppression uses the narrowest
  practical scope and `#[allow(clippy::lint_name, reason = "why this is safe")]`.
  Crate-wide or category-wide allowances require a dated specification decision.
- Prefer owned immutable values and explicit state transitions. Interior
  mutability, global state, and dynamic typing require a demonstrated boundary
  need and tests for lifecycle and concurrency.

### Errors and diagnostics

- `LiveError` and subordinate enums distinguish protocol, validation,
  authentication, authorization, CSRF, snapshot, revision, render, morph,
  provider, cache, upload, compatibility, and internal failures.
- Error conversion preserves a stable machine category, safe recovery
  instruction, source context, and causal chain. Do not flatten errors to strings
  or use status codes as the only taxonomy.
- Production messages contain no snapshot bytes, signatures, cookies, tokens,
  transient models, private HTML, SQL values, stack traces, or policy internals.
- Developer diagnostics point to the Rust declaration, template path and source
  region, directive, component, island, lifecycle phase, and correlation ID when
  available.

### Async execution and concurrency

- Tokio tasks are structured and cancellation-aware. Detached tasks require an
  explicit owner, shutdown path, bounded queue, and observability.
- Never hold a blocking mutex, database transaction, provider lease, or mutable
  component borrow across an unrelated `.await`.
- Every queue, batch, upload, body, parser, recursion depth, retry, lease,
  connection, and diagnostic buffer has a configured bound.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eas4ai/suprnova](https://github.com/eas4ai/suprnova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
