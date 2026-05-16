---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CentralGauge is an open-source benchmark for evaluating LLMs on AL (Application Language) code generation, debugging, and refactoring for Microsoft Dynamics 365 Business Central. The system provides two-attempt task execution with automated compilation and testing inside isolated BC containers.

## Memory

- Current year is 2026; today's date is the source of truth for "recent" model releases.
- Don't hardcode model IDs in code. Use the catalog (`site/catalog/models.yml`) or
  `deno task start models -p <provider> --live` to discover current names.
  Verify availability with `deno task start models <slug> --check` before running benchmarks.
- Container infra failures (SYSLIB0014, OOM, publish timeout, PSSession loss, container offline)
  auto-classify via `src/health/`. The bench dashboard shows a sticky red banner naming the
  signature + fix hint when a container hits the persistent-failure threshold (3-of-window same
  fingerprint). Phase A only — no auto-quarantine yet. After a fix, no `doctor containers`
  command exists; just restart the bench. Scores file gets a `# Container Health` block per run.

## Technology Stack

- **Runtime**: Deno 1.44+ with TypeScript 5
- **CLI Framework**: Cliffy Command (https://cliffy.io/docs@v0.25.4/command) - Use this for CLI argument parsing instead of manual parseArgs
- **Container**: bccontainerhelper + Windows NanoServer LCOW
- **Manifest**: YAML 1.2 format for task definitions
- **Reports**: JSON (machine-readable) and HTML (human-readable) with SvelteKit static generation
- **CI/CD**: GitHub Actions with Docker layer caching

## Environment

- We use Git Bash for shell commands, but use full Windows paths (e.g., `U:\Git\CentralGauge\src\file.ts`) in tool calls (Read, Edit, Write, Glob, Grep).
- `jq` is available for debugging and inspecting JSON files.

## Local BC Container

- Available containers: `Cronus28`, `Cronus281`, `Cronus282`, `Cronus283`, `Cronus284`, `Cronus285`
  (use `--containers Cronus28,Cronus281` for parallel compile/test)
- Credentials: `sshadows` / `1234`
- Health check URL: `http://Cronus28/BC/?tenant=default` (check if login page loads to verify container is up)

## bccontainerhelper config quirks

- Pinned to **6.1.11** in `bc-container-provider.ts` + `bc-script-builders.ts`
  (6.1.12+ disables PSSession for BC v28+ by default — breaks publish flow).
- `$bcContainerHelperConfig.usePwshForBc24 = $false` is REQUIRED in the bench's
  PowerShell scripts. With `$true`, the cached PSSession loses
  `Get-NavServerInstance` after any `Unpublish-BcContainerApp`, and the next
  `Publish-BcContainerApp` crashes. Don't flip without reproducing the
  multi-unpublish test sequence.

## Project Structure

| Directory | Purpose                                                                      |
| --------- | ---------------------------------------------------------------------------- |
| `cli/`    | CLI commands (Cliffy), helpers, TUI                                          |
| `src/`    | Core library (LLM adapters, container providers, task execution)             |
| `tests/`  | Unit and integration tests mirroring `src/` structure                        |
| `tasks/`  | Task YAML definitions organized by difficulty (`easy/`, `medium/`, `hard/`)  |
| `mcp/`    | MCP server for AL tools                                                      |
| `docs/`   | Architecture documentation                                                   |
| `site/`   | SvelteKit Cloudflare Worker scoreboard (D1 + R2) — see Ingest Pipeline below |

Key modules in `src/`:

- `llm/` - LLM adapters with registry and pooling
- `container/` - BC container providers with auto-detection
- `tasks/` - Task execution and transformation
- `parallel/` - Parallel execution orchestration
- `config/` - Configuration loading and merging
- `rules/` - Markdown rules generation from shortcomings
- `ingest/` - Bench → scoreboard payload, Ed25519 signing, R2 blob upload
- `errors.ts` - Structured error hierarchy

## Ingest Pipeline & Site

Bench results auto-ingest to the production scoreboard at
`https://centralgauge.sshadows.workers.dev` (Cloudflare Worker + D1 + R2).
Disable with `--no-ingest`.

- **Canonical site URL is `https://ai.sshadows.dk`** (custom-domain cutover ed13869). The workers.dev URL is internal-only — keep it out of public site content, tests, and source-level fallbacks. `SITE_BASE_URL` in `wrangler.toml` is the source of truth at runtime; `site/src/lib/shared/site.ts` holds the build-time fallback.
- `site/` — SvelteKit Worker. D1 schema in `site/migrations/`, API under `/api/v1/*`
- `src/ingest/` — payload builder, Ed25519 signer, R2 blob uploader, HTTP client w/ backoff
- `centralgauge ingest <results-file>` — manually replay a saved run
- `centralgauge sync-catalog --apply` — reconcile `site/catalog/*.yml` ↔ D1 catalog tables
- Config (URL, keys, machine_id) merged from `.centralgauge.yml` (cwd + home)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SShadowS/CentralGauge](https://github.com/SShadowS/CentralGauge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
