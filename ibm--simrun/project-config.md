---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core principles

Rule 1 — Think Before Coding.
- No silent assumptions.
- State what you're assuming.
- Surface tradeoffs.
- Ask before guessing.
- Push back when a simpler approach exists.

Rule 2 — Simplicity First.
- Minimum code that solves the problem.
- No speculative features.
- No abstractions for single-use code.
- If a senior engineer would call it overcomplicated — simplify.

Rule 3 — Surgical Changes.
- Touch only what you must.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor what isn't broken.
- Match existing style.

Rule 4 — Goal-Driven Execution.
- Define success criteria.
- Loop until verified.
- Don't tell Claude what steps to follow, tell it what success looks like and let it iterate.

Rule 5 — Use the model only for judgment calls
- Use Claude for: classification, drafting, summarization, extraction from unstructured text.
- Do NOT use Claude for: routing, retries, status-code handling, deterministic transforms.
- If a status code already answers the question, plain code answers the question.

Rule 6 — Surface conflicts, don't average them
- If two existing patterns in the codebase contradict, don't blend them.
- Pick one (the more recent / more tested), explain why, and flag the other for cleanup.
- "Average" code that satisfies both rules is the worst code.

Rule 7 — Read before you write
- Before adding code in a file, read the file's exports, the immediate caller, and any obvious shared utilities.
- If you don't understand why existing code is structured the way it is, ask before adding to it.
- "Looks orthogonal to me" is the most dangerous phrase in this codebase.

Rule 8 — Tests verify intent, not just behavior
- Every test must encode WHY the behavior matters, not just WHAT it does.
- A test like `expect(getUserName()).toBe('John')` is worthless if the function takes a hardcoded ID.
- If you can't write a test that would fail when business logic changes, the function is wrong.

Rule 9 — Checkpoint after every significant step
- After completing each step in a multi-step task: summarize what was done, what's verified, what's left.
- Don't continue from a state you can't describe back to me.
- If you lose track, stop and restate.

Rule 10 — Match the codebase's conventions, even if you disagree
- If the codebase uses snake_case and you'd prefer camelCase: snake_case.
- If the codebase uses class-based components and you'd prefer hooks: class-based.
- Disagreement is a separate conversation. Inside the codebase, conformance > taste.
- If you genuinely think the convention is harmful, surface it. Don't fork it silently.

Rule 11 — Fail loud
- If you can't be sure something worked, say so explicitly.
- "Migration completed" is wrong if 30 records were skipped silently.
- "Tests pass" is wrong if you skipped any.
- "Feature works" is wrong if you didn't verify the edge case I asked about.
- Default to surfacing uncertainty, not hiding it.

## Project Overview

ASP (Attack Simulation Platform) is a detection testing framework. It detonates attack simulations and verifies that expected security alerts are generated in target platforms (Elastic Security, Datadog). The single shipped binary is `simrun`, a web server with a SvelteKit frontend backed by Postgres.

## Build and Development Commands

This project uses [mise](https://mise.jdx.dev/) for tool management.

```bash
# Build the simrun binary (frontend + server)
mise run build

# Build only frontend
mise run build-frontend

# Regenerate parser from JSON schemas
mise run parser

# Generate mocks (uses mockery)
go generate ./...

# Run all tests
go test ./...

# Run a single test
go test -v ./simrun/internal/... -run TestName

# Lint and format (golangci-lint, pinned in mise.toml)
mise run lint
mise run fmt
```

## Architecture

### Core Components

**Detonators** (`simrun/internal/detonators/`) - Execute attack simulations:
- `SimrunDetonator` - Detonate using simulation packs (Terraform-based)
- `AWSCLIDetonator` - Execute AWS CLI commands

**Injectors** (`simrun/internal/injectors/`) - Inject logs directly into SIEM:
- `ElasticInjector` - Inject documents into Elasticsearch

**Alert Matchers** (`simrun/internal/matchers/`) - Verify expected alerts:
- `elastic/` - Match Elastic Security Detection alerts
- `datadog/` - Match Datadog security signals

**Collectors** (`simrun/internal/collectors/`) - Collect logs after detonation:
- `ElasticCollector` - Collect related logs from Elasticsearch

**Parser** (`simrun/internal/parser/`) - Parse YAML scenario files into Scenario objects. Code is generated from JSON schemas via `mise run parser`. ParseOptions takes `Packs []config.PackConfig` plus run-scoped `EnvVars`, `DataDir`, `TerraformVersion`, `PackLogsEnabled`.

**Config** (`simrun/internal/config/`) - Type-only package with `Bootstrap` (env-only deploy config), `AppConfig` (DB-backed admin defaults), and `PackConfig`/`PackType` (in-memory parser/runner shapes). No more singleton.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/simrun](https://github.com/IBM/simrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
