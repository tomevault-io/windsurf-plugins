---
trigger: always_on
description: Guidance for coding agents and contributors working in this repository. Keep
---

# AGENTS.md

Guidance for coding agents and contributors working in this repository. Keep
changes small, tested, and consistent with the patterns already present.
`CLAUDE.md` is a symlink to this file so project guidance has one source of
truth.

## Project overview

Decant is a local-first Bun and TypeScript application that turns Claude Code
and Codex JSONL session logs into a normalized, full-text-searchable SQLite
archive. The `decant` CLI owns parsing, ingest, reads, reports, distillation,
recommendations, watch mode, and the React UI served by `decant serve`.

Start with these files and guides:

- `src/cli.ts` handles command composition, output, and exit-code policy.
- `src/sources/` contains source parsers and the primary extension point.
- `src/db.ts`, `src/schema.sql`, and `src/schema-manifest.ts` manage storage
  and schema.
- `src/server.ts` and `src/ui/` serve the local API and web UI.
- `docs/architecture.md` maps module boundaries and data flow.
- `docs/analytics-methodology.md` details metric definitions.
- `docs/data-lifecycle.md` describes sync and local state.
- `docs/api/openapi.yaml` defines the local API contract.
- `docs/distribution.md` covers npm, installer, Homebrew, Docker, and source
  builds.
- `docs/adding-a-source.md` provides the parser contribution checklist.

## Setup and commands

Run commands from the repository root. Bun 1.3 or newer is required. Docker is
only needed when validating the container image.

- `bun run dev` performs a frozen dependency install, startup sync, local UI,
  and source watcher.
- `bun run src/cli.ts --help` runs the CLI from source.
- `bun test` runs the test suite.
- `bunx tsc --noEmit` runs the type check.
- `bunx biome check .` runs the formatting and lint check.
- `just check` runs the full local gate, including the native distribution
  staging smoke. It needs network access because the smoke packs and installs
  the npm launcher packages.

The default archive is `~/.decant/decant.db`. Override it with `DECANT_DB` or
`--db`. Set `DECANT_NO_SYNC` or pass `--no-sync` whenever a command points at a
scratch archive, or automatic sync may populate it from the real Claude Code
and Codex directories.

## Definition of done

A change is ready when:

- `bun test` passes.
- `bunx tsc --noEmit` passes.
- `bunx biome check .` passes.
- New behavior has focused tests. Do not weaken or delete tests to make a
  change pass.
- Distribution changes get a native binary smoke test. Docker changes also get
  a local `docker build` and `docker run --help` smoke when Docker is
  available.
- User-visible command, route, configuration, or metric changes update their
  corresponding public docs.

## Project invariants

1. **Core modules stay print-free.** Parsing, ingest, query, stats, distill,
   recommendations, and database helpers return data or structured errors.
   Human-readable CLI output and exit codes belong in `src/cli.ts`.
2. **One process owns SQLite.** The CLI process opens the archive directly. WAL
   mode allows reads and ingest to coexist, so do not add another process that
   opens the database behind a separate contract.
3. **Runtime stays local-first.** Do not add outbound network calls, hosted
   service dependencies, or LLM calls. The only runtime networking is the local
   UI and API served by `decant serve`, bound to loopback by default.
4. **Costs are computed at ingest.** `estimateCost` in `src/cost.ts` is applied
   when a session is written. Pricing changes do not rewrite historical rows,
   so users must rebuild the archive to recompute them.
5. **Schema constants are the source of truth.** Use `LATEST_SCHEMA_VERSION` in
   `src/db.ts` rather than copying the current number into documentation.
   Unsupported older archives are rebuild-only. A migration is frozen once
   committed because someone may already have opened an archive with that
   branch, so put later schema adjustments in a new migration. Bump
   `INGEST_PIPELINE_REVISION` whenever unchanged sources must be re-derived so
   the next sync backfills them once and later syncs stay idempotent.
6. **Parsers are the extension point.** A new source requires
   `src/sources/<tool>.ts`, hand-written synthetic fixtures, parser tests,
   ingest/query coverage, and reviewed golden updates. Follow
   `docs/adding-a-source.md`.
7. **Serve routes are a documented local API.**
   `docs/api/openapi.yaml` is the source contract and `/api/openapi.json` is its
   runtime representation. Keep the implementation, OpenAPI document, route
   guide, and contract tests in sync.

## Security and privacy

- Never commit secrets, API keys, tokens, `.env` files, private keys, real
  transcripts, exports from real sessions, or personal archive databases.
- `~/.claude` and `~/.codex` contain private content. Fixtures must be written
  from scratch, and `test/golden/` must be generated only from those synthetic
  fixtures.
- Keep source parsing resilient, meaning malformed records should become
  diagnostics, not crashes or leaked transcript content.
- Non-loopback serving must require explicit trusted-peer configuration. Treat
  `Host` and browser-origin checks as defense in depth, not authentication.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dosu-ai/decant](https://github.com/dosu-ai/decant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
