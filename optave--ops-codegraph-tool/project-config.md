---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Hooks enforce code quality.** This project uses Claude Code hooks (`.claude/hooks/`) to automatically inject file-level dependency context on reads, rebuild the graph after edits, block commits with cycles or dead exports, run lint on staged files, and show diff-impact before commits. If codegraph reports an error or produces wrong results when analyzing itself, **that's a real bug** — don't work around it or ignore it. Flag it to the user and, if it's blocking the current task, fix it.

> **Never fabricate facts.** Do not state licenses, version numbers, feature claims, or any factual information without first verifying it (read the file, run the command, check the source). If you don't know, say so — do not guess.

> **Never document bugs as expected behavior.** If two engines (native vs WASM) produce different results, that is a bug in the less-accurate engine — not an acceptable "parity gap." Adding comments or tests that frame wrong output as "expected" blocks future agents from ever fixing it. Instead: identify the root cause, file an issue, and fix the extraction/resolution layer that produces incorrect results. The correct response to "engine A reports 8 cycles, engine B reports 11" is to fix the 3 false cycles in engine B, not to document why the difference is okay.

> **Never silently skip verification.** If tests, builds, or any verification step cannot run or fails for any reason (compilation errors, platform issues, missing dependencies), STOP and report the issue to the user immediately. Never silently proceed with unverified changes. Let the user decide whether to proceed — do not make that decision yourself.

## Codegraph Workflow

Hooks handle: file-level deps on reads, graph rebuild after edits, commit-time checks (cycles, dead exports, diff-impact, lint). **Use these for function-level understanding when modifying source code:**

### Before modifying code:
Pick the commands that fit the situation — don't run all four mechanically:
- `codegraph where <name>` — find where the symbol lives
- `codegraph audit --quick <target>` — understand the structure
- `codegraph context <name> -T` — get full context (source, deps, callers)
- `codegraph fn-impact <name> -T` — check blast radius before editing

Skip the above commands for non-code files, trivial edits, or when you already have sufficient context.

### After modifying code:
- `codegraph diff-impact --staged -T` — verify impact before committing

### Navigation
- `codegraph where --file <path>` — file inventory (symbols, imports, exports)
- `codegraph query <name> -T` — function call chain (callers + callees)
- `codegraph path <from> <to> -T` — shortest call path between two symbols
- `codegraph exports <file> -T` — per-symbol export consumers
- `codegraph children <name> -T` — sub-declarations (parameters, properties, constants)
- `codegraph search "<query>"` — semantic search (requires `codegraph embed`)
- `codegraph ast --kind call <name> -T` — find all call sites of a function

### Impact & analysis
- `codegraph diff-impact main -T` — impact of branch vs main
- `codegraph audit <target> -T` — structural summary + impact + health in one report
- `codegraph triage -T` — ranked audit priority queue
- `codegraph complexity -T` — per-function complexity metrics
- `codegraph batch t1 t2 t3 -T --json` — batch query multiple targets

### Overview & health
- `codegraph map` — module overview (most-connected files)
- `codegraph stats` — graph health and quality score
- `codegraph structure --depth 2` — directory tree with cohesion scores
- `codegraph roles --role dead -T` — find dead code (unreferenced symbols)
- `codegraph roles --role core -T` — find core symbols (high fan-in)
- `codegraph branch-compare main HEAD -T` — structural diff between refs

### Flags
- `-T` — exclude test files (use by default) · `-j` — JSON output
- `-f, --file <path>` — scope to file · `-k, --kind <kind>` — filter kind

## Project Overview

Codegraph (`@optave/codegraph`) is a local code dependency graph CLI. It parses codebases with tree-sitter (WASM), builds function-level dependency graphs stored in SQLite, and supports semantic search with local embeddings. No cloud services required.

**Languages supported (23):** JavaScript, TypeScript, TSX, Python, Go, Rust, Java, C#, Ruby, PHP, C, C++, Kotlin, Swift, Scala, Bash, Elixir, Lua, Dart, Zig, Haskell, OCaml, Terraform/HCL. `LANGUAGE_REGISTRY` in `domain/parser.ts` is the single source of truth — check there for the current list.

## Commands

```bash
npm install                      # Install dependencies
npm test                         # Run all tests (vitest)
npm run test:watch               # Watch mode
npm run test:coverage            # Coverage report
npx vitest run tests/parsers/javascript.test.ts   # Single test file
npx vitest run -t "finds cycles"                  # Single test by name
npm run build:wasm               # Rebuild WASM grammars from devDeps (built automatically on npm install)
```

**Linter/Formatter:** [Biome](https://biomejs.dev/) — config in `biome.json`, scoped to `src/` and `tests/`.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [optave/ops-codegraph-tool](https://github.com/optave/ops-codegraph-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
