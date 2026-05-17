---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

The user-facing entry point is [README.md](README.md). Deeper material is split across `docs/`:
- [docs/architecture.md](docs/architecture.md) — crate graph, state machine, workflow engine, DB.
- [docs/api.md](docs/api.md) — REST surface, pagination, MCP tools.
- [docs/getting-started.md](docs/getting-started.md) — install, configuration, end-to-end walkthrough.
- [docs/cli.md](docs/cli.md) — `forge-ctl` reference.

When you change behavior, update the relevant `docs/` file in the same change. Treat README as a launch landing page — keep it short and link out.

## Project status: Public Beta (`0.1.x`)

Forge is in public beta. The local-first single-user product is shipping to real users, but we are still pre-`1.0` and pre-stable.

What that means for changes you make:

- **Breaking changes are allowed**, but they must be **visible**: update [CHANGELOG.md](CHANGELOG.md) under the `Unreleased` section with a `### Breaking` entry, and bump the workspace version when cutting a release. Silent breakage is no longer acceptable now that users exist.
- **No backward-compat shims, no `_v2` suffixes, no deprecated aliases.** When something changes, update all call sites and delete the old code. Express the break in the changelog, not in the source.
- **No feature flags for rollout safety.** Ship the new behavior directly.
- **Database migrations must preserve user data** unless the user explicitly opts in to a destructive migration (e.g. a hard schema reset for the dev `./test` data dir). Add a new numbered migration; do not edit historical ones.
- **Public surfaces** — REST endpoints, MCP tool names, `forge-ctl` flag names, event payload shapes, JSONL log schema, on-disk file layout — are the ones to be most deliberate about. Internal Rust APIs and crate boundaries can still move freely.
- When a spec/design changes, rewrite the affected code to match. Don't layer the new design on top of the old one.

If you find yourself writing `// kept for backward compatibility`, a `_v2` suffix, a re-export shim, or a deprecated alias — stop, just change the code, and note the break in the changelog.

A stable `1.0` will land once the workflow engine consolidation, multi-user story, and release artifact hardening (signing, SBOMs, Homebrew, Windows builds) are finalized. Until then, assume the public surface is still moving.

## Build & test commands

```bash
cargo build                          # Build entire workspace
cargo test                           # Run all tests
cargo test -p db                     # Run tests for a specific crate
cargo test -p api --test happy_path  # End-to-end happy-path test
cargo run -p forge-client -- --help  # forge-ctl CLI client
cargo build -p forge-client          # Build forge-ctl binary
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

The happy-path test (`crates/api/tests/happy_path.rs`) is the canonical end-to-end smoke and a forcing function for spec alignment — if you break it, you probably need to revisit the spec, not the test.

### Running the server

Use the Makefile targets for local dev and manual/agent testing — they all point data at `./test/` (gitignored) instead of `~/.forge/`, so test state never pollutes real user data:

```bash
make dev               # start server → ./test data dir (default for testing)
make dev-demo          # same + seed demo data (idempotent)
make dev-no-daemon     # server only, no embedded AI daemon
make frontend          # Vite dev server on :5173, proxies /api to :8080
make clean-test        # wipe ./test directory and start fresh
```

Direct invocations (when you need flags not covered by `make`):

```bash
cargo run -p forge-cli -- --data-dir ./test           # explicit local data dir
cargo run -p forge-cli -- --no-mcp                    # disable MCP endpoint
FORGE_DATA_DIR=./test cargo run -p forge-cli          # env-var equivalent
# Server always binds to 127.0.0.1:8080
```

### Frontend

```bash
cd web && pnpm install && pnpm dev    # Vite dev server on :5173, proxies /api to :8080
cd web && pnpm build                  # Production build to web/dist/
cd web && pnpm lint && pnpm typecheck && pnpm test
```

`pnpm` is the package manager (enforced by `pnpm-lock.yaml`). Don't introduce `npm` or `yarn` lockfiles.

## When working on this repo

- **Read `docs/architecture.md` before changing service wiring, the workflow engine, or the task state machine.** That doc is the source of truth for the lifecycle, hook ordering, role assignment, and retry-budget rules.
- **Public-API changes touch four places**: the route handler in `crates/api/src/routes/`, the request/response type in `crates/api-types/`, the generated TS types under `web/src/types/generated/`, and `docs/api.md`. Update all four in one change.
- **State-machine changes** must keep `crates/api/tests/happy_path.rs` green and the table in `docs/architecture.md#task-state-machine` accurate.
- **Migrations** are numbered `V{NNN}__{name}.sql`. Add a new file; don't edit historical migrations even during beta — users have running databases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ForgeAILab/forge](https://github.com/ForgeAILab/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
