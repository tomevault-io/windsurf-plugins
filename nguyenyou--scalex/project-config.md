---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Scalex

Scalex is a Scala code intelligence CLI for coding agents. It provides fast symbol search, find definitions, and find references — without requiring an IDE, build server, or compilation. Designed as a Claude Code plugin.

## IMPORTANT: No company references

NEVER mention any company names, internal project names, proprietary codebases, or organization-specific details in any output — including commit messages, PR descriptions, changelogs, roadmaps, documentation, code comments, or conversations. Always use generic examples (e.g. `HttpMessageService`, `UserServiceLive`) instead.

## Search scope

Source code lives in `src/` (production) and `tests/` (test suite). When searching for Scala code, scope searches to these directories. Avoid searching repo-wide — `benchmark/` contains ~17.7k Scala files from the scala3 compiler clone that will pollute results.

## Workflow

- Before planning or implementing any feature, first add it to `docs/ROADMAP.md` under the appropriate section
- The roadmap is the source of truth for what's planned and what's done
- **Bug fix workflow**: When receiving a bug report, always write a failing test that reproduces the bug *before* writing the fix. This validates the bug is real and ensures the fix is verifiable. Only then apply the code fix and confirm the test passes.

## Build & Run

```bash
# Run via scala-cli (development)
scala-cli run src/ -- <command> [args...]

# Run tests
scala-cli test src/ tests/

# Build GraalVM native image (requires GraalVM + scala-cli)
./build-native.sh
# Output: ./scalex (26MB standalone binary)

# Validate Claude Code plugin structure
claude plugin validate plugins/scalex/
```

## Architecture

Source code is in `src/`, tests in `tests/` (Scala 3.8.2, JDK 21+). When searching the codebase, scope to these directories to avoid hitting benchmark data or build artifacts.

```
src/                           # Production source code
├── project.scala              # scala-cli directives only
├── model.scala                # Data types, enums, version constant
├── extraction.scala           # AST parsing & single-file extraction functions
├── index.scala                # Git integration, persistence, WorkspaceIndex, filtering
├── analysis.scala             # Cross-index analysis (hierarchy, overrides, deps, diff, ast-pattern)
├── format.scala               # Text formatters for symbols and references
├── cli.scala                  # Arg parsing, workspace resolution, @main entry point
├── command-helpers.scala      # Shared filters: filterSymbols, filterRefs, mkNotFoundWithSuggestions
├── dispatch.scala             # Command map + runCommand
└── commands/                  # One file per command (ls to discover all 25 commands)
    ├── definition.scala       # cmdDef
    ├── search.scala           # cmdSearch
    ├── refs.scala             # cmdRefs
    ├── hierarchy.scala        # cmdHierarchy
    ├── overview.scala         # cmdOverview
    └── ...                    # 25 commands total

tests/                         # Test suite
├── test-base.test.scala       # Shared test fixture (workspace setup)
├── extraction.test.scala      # Extraction tests
├── index.test.scala           # Index/search/persistence tests
├── analysis.test.scala        # Analysis tests (hierarchy, overrides, deps, etc.)
└── cli.test.scala             # CLI/formatting/command output tests

benchmark/                     # Benchmark data (gitignored)
├── scala3/                    # Shallow clone of scala/scala3 for benchmarks
└── results/                   # Hyperfine JSON exports
```

### Pipeline

```
git ls-files --stage → Scalameta parse → in-memory index → query
                              ↓
                    .scalex/index.bin (binary cache, OID-keyed, bloom filters)
```

1. **Git discovery**: `git ls-files --stage` returns all tracked `.scala` files with their OIDs
2. **Symbol extraction**: Scalameta parses source ASTs (Scala 3 first, falls back to Scala 2.13), extracts top-level symbols (class/trait/object/def/val/type/enum/given/extension)
3. **OID caching**: On subsequent runs, compares OIDs — skips unchanged files entirely
4. **Persistence**: Binary format with string interning at `.scalex/index.bin`
5. **Bloom filters**: Per-file bloom filter of identifiers — `refs` and `imports` only read candidate files

### Code style

- **Named tuples**: Never use unnamed tuples. Whenever a tuple is needed — return types, local variables, collection elements — always use named tuples. E.g. `(results: List[Reference], timedOut: Boolean)` not `(List[Reference], Boolean)`.
- **No `return` statements**: Never use `return` anywhere — not in methods, not in lambdas, not in `for`/`foreach`. Use `scala.util.boundary` + `boundary.break` for early exit, or restructure with `match`/`if-else`. The `return` keyword is deprecated in Scala 3 inside lambdas and is a footgun everywhere else. Existing `return` statements in the codebase are legacy — do not add new ones, and remove them when touching nearby code.

### Key design choices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nguyenyou/scalex](https://github.com/nguyenyou/scalex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
