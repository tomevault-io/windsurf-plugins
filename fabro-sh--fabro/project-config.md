---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and test commands

### Rust
- `cargo build --workspace` — build all crates
- `cargo nextest run --workspace` — run all unit tests
- `cargo nextest run -p fabro-server` — test a single crate
- `cargo nextest run -p fabro-workflow -- test_name` — run a single test
- `set -a && source .env && set +a && cargo nextest run --workspace --profile e2e --run-ignored only` — run all E2E live tests (requires credentials in `.env`, see `.env.example`)
- `set -a && source .env && set +a && cargo nextest run -p fabro-llm --profile e2e --run-ignored only` — run E2E tests for a single crate
- `cargo +nightly-2026-04-14 fmt --check --all` — check formatting (pinned nightly required for rustfmt config; CI uses the same date)
- `cargo +nightly-2026-04-14 fmt --all` — auto-format
- `cargo +nightly-2026-04-14 clippy --workspace --all-targets -- -D warnings` — lint (CI runs nightly clippy to match; install with `rustup toolchain install nightly-2026-04-14 --profile minimal --component clippy,rustfmt`)

macOS note: if `cargo nextest run` fails with `Too many open files (os error 24)` / `EMFILE`, raise the shell's soft FD limit before running tests, for example `ulimit -n 4096 && cargo nextest run --workspace`. Some terminals and inherited agent sessions start with `ulimit -n 256`, which is too low for the shared CLI test daemon under parallel nextest load.

### TypeScript (fabro-web)
- `cd apps/fabro-web && bun run dev` — rebuild web assets on change for the Rust server; refresh the browser manually
- `cd apps/fabro-web && bun test` — run tests
- `cd apps/fabro-web && bun run typecheck` — type check
- `cd apps/fabro-web && bun run build` — production build (writes to `apps/fabro-web/dist/` only; does NOT update the bundled SPA that ships in the Rust binary)
- `scripts/refresh-fabro-spa.sh` — **run this before committing any TypeScript change in `apps/fabro-web/` or `lib/packages/fabro-api-client/`**. It runs the production build and then copies `dist/` into `lib/crates/fabro-spa/assets/` (which is tracked in git). CI's TypeScript `Build` job reruns this script and then `git diff --exit-code -- lib/crates/fabro-spa/assets` — if the committed bundle drifts from source (e.g. content-hashed filenames like `entry-<hash>.js` change), the check fails. `bun run build` on its own is not enough.

### Docker image
- `bin/dev/docker-build.sh` — builds the local Docker image from the current tree using the release pipeline's cargo-zigbuild approach. Honors `--arch amd64|arm64`, `--tag <name>` (default `fabro`), and `--compile-only` (stages `docker-context/<arch>/fabro` without `docker build`). Prefer this over writing a throwaway Dockerfile; the release pipeline, `Dockerfile`, and this script share the same binary layout.
- Refresh the embedded SPA before rebuilding the image after any `apps/fabro-web` change: `scripts/refresh-fabro-spa.sh` runs the bun build and copies `dist/` into `lib/crates/fabro-spa/assets/`. Skipping this step produces a Docker image whose Rust binary embeds a stale SPA bundle.

### Marketing site (apps/marketing)
- `cd apps/marketing && bun run dev` — start Astro dev server
- `cd apps/marketing && bun run build` — production build
- `cd apps/marketing && bunx vercel --prod` — deploy to Vercel (project: website, domain: fabro.sh)

### Dev servers
1. `fabro server start` — starts the Rust API server (demo mode is per-request via `X-Fabro-Demo: 1` header)
2. `cd apps/fabro-web && bun run dev` — rebuilds web assets on change; refresh the browser manually
3. Mintlify docs dev server (requires Docker — `mintlify dev` needs Node LTS which may not match the host):
   ```
   docker run --rm -d -p 3333:3333 -v $(pwd)/docs:/docs -w /docs --name mintlify-dev node:22-slim \
     bash -c "npx mintlify dev --host 0.0.0.0 --port 3333"
   ```
   Then open http://localhost:3333. Stop with `docker stop mintlify-dev`.

## API workflow

The OpenAPI spec at `docs/api-reference/fabro-api.yaml` is the source of truth for the fabro-api HTTP interface.

1. Edit `docs/api-reference/fabro-api.yaml`
2. `cargo build -p fabro-api` — build.rs regenerates Rust types and client via progenitor
3. Write/update handler in `lib/crates/fabro-server/src/server.rs`, add route to `build_router()`
4. `cargo nextest run -p fabro-server` — conformance test catches spec/router drift
5. `cd lib/packages/fabro-api-client && bun run generate` — regenerates TypeScript Axios client

## Architecture

Fabro is an AI-powered workflow orchestration platform. Workflows are defined as Graphviz graphs, where each node is a stage (agent, prompt, command, conditional, human, parallel, etc.) executed by the workflow engine.

### Rust crates (`lib/crates/`)
- **fabro-cli** — CLI entry point. Commands: `run`, `exec`, `serve`, `validate`, `parse`, `cp`, `model`, `doctor`, `install`, `ps`, `system prune`
- **fabro-workflow** — Core workflow engine. Parses Graphviz graphs, runs stages, manages checkpoints/resume, hooks, retros, and human-in-the-loop interactions
- **fabro-agent** — AI coding agent with tool use (Bash, Read, Write, Edit, Glob, Grep, WebFetch). `Sandbox` trait abstracts execution environments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabro-sh/fabro](https://github.com/fabro-sh/fabro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
