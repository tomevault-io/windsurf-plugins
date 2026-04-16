---
trigger: always_on
description: CLI tool that benchmarks Claude Code audit skills against each other amd/or a bare Claude  Code baseline/bare. Runs multiple iterations of each condition against real Solidity codebases, captures raw output, verifies execution integrity, and generates comparison reports with recall/FP scoring against ground truth.
---

# CLAUDE.md — Benchmark Runner

## What This Is

CLI tool that benchmarks Claude Code audit skills against each other amd/or a bare Claude  Code baseline/bare. Runs multiple iterations of each condition against real Solidity codebases, captures raw output, verifies execution integrity, and generates comparison reports with recall/FP scoring against ground truth.

## Setup Requirements

- Node 20+
- `claude` CLI installed (all LLM calls use the CLI binary via `child_process.spawn` — no Anthropic SDK)
- **Exclude `results/` and `workspaces/` from cloud sync** (Dropbox, iCloud, OneDrive). File sync during benchmark writes causes data corruption — see [Platform Limitations](docs/persisted-output-trap.md). Run `npm run setup` to create `.nosync` markers.

## Quickstart

```bash
# 0. First-time setup (creates .nosync markers for sync protection)
npm run setup

# 1. Run benchmark
npm run bench -- --codebases merkl-stripped --runs 3 --parallel

# 2. Analyze results (classify → cluster → validate → report)
npm run analyze

# 3. Read the results
cat summary.md

# 4. Management dashboard (optional, no LLM calls)
npm run dashboard
open dashboard.html
```

## Project Structure

```
src/
  shared/           Types, parser, utilities — shared across all modules
    types.ts        Central type definitions
    report-data.ts  Shared data feed (ReportData) — single source of truth for all renderers
    parser.ts       Finding extraction from raw audit output
    util/           Logger, shell helpers, hash utility
  runner/           Benchmark execution — spawns claude, captures output
    cli.ts          CLI entrypoint (npm run bench)
    config.ts       Config loader (bench.json)
    runner.ts       Main benchmark loop
    workspace.ts    Workspace creation (real copies per condition)
    skill.ts        Skill version management
    verify.ts       Post-run verification (12 checks)
  classifier/       Analysis pipeline — independent from runner
    llm.ts          LLM call utility with injectable LLMProvider (CLI default, fake for tests)
    classify.ts     GT classification with configurable Nx majority vote
    cluster.ts      Novel finding clustering (incremental + chunked full)
    validate.ts     Novel validation against scoped source code
    pipeline.ts     Orchestrator: classify → cluster → validate
    pipeline-cli.ts CLI entrypoint (npm run analyze)
  reports/          Report generation — consumes shared data feed
    report.ts       Markdown renderer (deterministic tables + LLM narrative)
    report-cli.ts   CLI entrypoint (npm run report)
  dashboard/        Management HTML dashboard — consumes same shared data feed
    render.ts       HTML renderer (npm run dashboard)
  archive/          Result archival with provenance manifest
    archive.ts      Core archive logic (npm run archive)
config/             JSON benchmark configs (bench.json)
datasets/           Solidity codebases to audit (submodules + canary inline)
skills_versions/    Pinned skill snapshots (pashov/, darknavy/, each with source.json provenance)
workspaces/         Ephemeral real-copy workspaces (gitignored, auto-cleaned)
results/            Run outputs + report-data.json shared feed (gitignored)
ground_truth/       Known-bug answer keys per codebase (JSON, enables Recall/FP scoring)
ground_truth/reports/  Official C4 audit reports (markdown)
docs/               Isolation strategy, benchmark prompt template, rewrite plan, testing strategy
archive-results/    Archived result snapshots with MANIFEST.json (gitignored)
tests/              Vitest test suite (275 tests, 17 files)
```

## How It Works

`npm run bench` runs the full matrix (all codebases x all conditions x N runs). **This is expensive** — prefer filtered runs during development.

1. Reads `config/bench.json`
2. Prepares workspaces (one per codebase × condition):
   - Each condition gets its own workspace: `workspaces/<codebase>__<conditionId>/`
   - Codebase files are **real copies** (not symlinks) from `datasets/`
   - A `CLAUDE.md` is written per workspace to block parent walk-up and deliver scope info
   - **Skill runs**: `.claude/commands/solidity-auditor/` installed with correct version + VERSION verified
   - **Bare runs**: no skill installed, just the codebase copy
3. For each `(codebase, condition, iteration)`:
   - Spawns `claude -p "<prompt>" --output-format stream-json --verbose` in the workspace cwd
   - All runs: `--dangerously-skip-permissions --setting-sources project,local`
   - Bare runs add `--disable-slash-commands --disallowedTools Skill`
   - 10-minute timeout with grace-kill (15s after `result` event → SIGTERM → SIGKILL)
   - Captures 4 files per run
4. Post-run verification: 12 automated checks per run
5. Cleans up all workspaces after suite completes

**Invalid run detection**: Skill runs that exit 0 but produce 0 findings or <500 chars stdout are marked `INVALID` and excluded from report averages. This catches corrupt runs (e.g., cloud sync inode detachment).

## Commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mishoko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
