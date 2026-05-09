---
trigger: always_on
description: lineark is an unofficial Linear (issue tracker) ecosystem for Rust:
---

# CLAUDE.md — lineark

## What is this

lineark is an unofficial Linear (issue tracker) ecosystem for Rust:
- **lineark-sdk** (`crates/lineark-sdk/`) — Async-first Rust SDK for the Linear GraphQL API
- **lineark-derive** (`crates/lineark-derive/`) — Proc macro crate providing `#[derive(GraphQLFields)]` for type-driven field selection
- **lineark** (`crates/lineark/`) — CLI for humans and LLMs, powered by lineark-sdk
- **lineark-codegen** (`crates/lineark-codegen/`) — Internal tool that reads `schema/schema.graphql` and generates typed Rust code into `crates/lineark-sdk/src/generated/`

See `docs/MASTERPLAN.md` for the full architecture, roadmap, and design decisions (huge file, only read if needed, to keep token consumption down in day-to-day work).

## Workspace layout

```
Cargo.toml              # workspace root
crates/
  lineark-sdk/          # library (published to crates.io)
  lineark-derive/       # proc macro for #[derive(GraphQLFields)]
  lineark/              # CLI binary (published to crates.io + binary releases)
  lineark-codegen/      # codegen tool (not published)
schema/
  schema.graphql        # Linear's GraphQL schema (checked in, fetched from API)
  operations.toml       # allowlist of which queries/mutations to generate
docs/
  MASTERPLAN.md         # full project plan and roadmap
```

## Key commands

```bash
rustup update stable                     # sync local toolchain with CI (do this before developing)
cargo run -p lineark-codegen             # regenerate SDK types from schema
cargo run -p lineark -- <args>           # run the CLI
make check                               # lint + doc + build (no tests)
make test                                # offline tests (unit + integration, fast, no API token)
make test-online                         # online tests (live API, serial, needs token)
```

`make check && make test` is the canonical local pre-push check. Add `make test-online` when you have a test API token and need full coverage.

**Online tests must run serially.** They hit the live Linear API, which has plan-level limits (e.g. max teams). Running them in parallel causes spurious failures from resource exhaustion. `make test-online` handles this automatically.

### Writing online tests: the three pillars

Linear's API has permanent, structural consistency issues — not "temporary flakiness that will go away". Treat them as part of the environment you're testing against. **Every online test must apply all three of these**, in every CRUD path it touches, or it will flake in CI:

**1. Proactive cleanup.** Never assume the workspace is empty, and never leak resources between tests.

- Call `create_test_team()` (from `lineark-test-utils`) for every test that needs a team. It invokes `cleanup_zombies()` once per process and assigns a fresh unique team key, dodging Linear's indexed-identifier collisions.
- Wrap every created resource in the matching RAII guard from `lineark_test_utils::guards` (`TeamGuard`, `IssueGuard`, `ProjectGuard`, `DocumentGuard`, `LabelGuard`) *immediately* after the create returns its ID. Guards delete on drop, so cleanup happens even when a later assert panics. Never rely on explicit `.delete(...)` calls — a panic will skip them.
- If the test creates resources through a path the guards don't yet cover (e.g. a new Linear entity type), add a new guard to `crates/lineark-test-utils/src/guards.rs` before shipping the test.

**2. Unique names per attempt.** Linear's API returns phantom "conflict on insert" errors keyed on request body content, so two attempts with the same body get the same sticky conflict.

- Generate a per-test random suffix: `format!("[test] <what> {}", &uuid::Uuid::new_v4().to_string()[..8])`. The `[test]` prefix is what `cleanup_workspace` looks for.
- If a helper retries a mutation on transient failure (see pillar 3), it **must mutate the request body between attempts** — not just re-send. Our CLI helper `run_lineark_with_retry` appends a fresh `retry-<uuid6>` suffix to the `<name>` positional and returns the actually-used name so the test can look the entity up later.
- Don't reuse fixture names across tests. Each test call site should get its own UUID-suffixed name.

**3. Retries on every API call that can fail transiently.**

- **Creates** (`retry_create` in `lineark-test-utils`): **15 attempts** with backoffs `[0, 2, 5, 10, 20, 30, 60×9]`s (~9 min worst case), retries on "conflict on insert" / "already exists". Use for any SDK `*_create` call. The CLI-shell variant is `run_lineark_with_retry` in `crates/lineark/tests/online.rs` — same 15-attempt budget, plus body mutation (appends a fresh `retry-<uuid6>` suffix to the positional after `"create"`, gated on `[test]` prefix so UUID positionals like `comments create <uuid>` aren't corrupted).
- **Reads after a recent create** (`retry_search` / `retry_with_backoff`): Linear's search/filter indexes are eventually consistent — a freshly-created resource may not be queryable by name for several seconds. Use `retry_search` with a predicate, or `retry_with_backoff` for arbitrary checks that need time to propagate.
- **`settle()` (5s sleep)** between a create and the subsequent assertion, when retry-with-predicate isn't applicable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipbit03/lineark](https://github.com/flipbit03/lineark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
