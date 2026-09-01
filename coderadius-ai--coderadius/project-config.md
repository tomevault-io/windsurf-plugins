---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## What This Is

CodeRadius is a CLI tool that builds an architectural knowledge graph from polyglot codebases. It statically analyzes source code (TypeScript, PHP, Go, Python), extracts infrastructure dependencies via LLM-driven semantic analysis, persists everything to a Neo4j/Memgraph graph database, and enables cross-repo impact analysis ("blast radius") and governance policy enforcement.

## Build & Run Commands

**Runtime:** Bun (not Node.js for development). The project uses ES modules (`"type": "module"`).

```bash
bun install                # Install dependencies
bun run build              # TypeScript compilation (tsc, noEmit)
bun run dev                # Run CLI directly: bun run src/cli/index.ts
bun run dev:dashboard      # Dashboard dev server with live reload (http://localhost:3456)
```

### Testing

```bash
bun run test:unit                                    # All unit tests
bun vitest run tests/unit/path/to/file.test.ts       # Single unit test
bun run test:integration                             # Integration tests (requires running Memgraph)
make test-eval-golden      # Eval tests (LLM golden + replay-cached patterns)
make test-patterns         # Pattern fixtures, deterministic subset (no LLM, no DB)
```

The repo organises tests by determinism level and dependency footprint. Pick the right tier when adding a feature or fixing a bug:

- **Unit tests** (`tests/unit/`): pure logic, no external services. Defaults for sanitizer rules, schema validation, regex guards, in-memory pipelines. Run: `bun run test:unit` (~7s).

- **Integration tests** (`tests/integration/`): exercise real graph mutations against Memgraph. Required for any change to `src/graph/mutations/`, the welder (`dynamic-infra-resolver.ts`), or DB-backed pipelines. Run sequentially (no file parallelism). Run: `bun run test:integration` (~20s).

- **Eval tests — agents** (`tests/eval/agents/`): LLM extraction quality on per-function snippets, replay-cached. Use when a fix changes prompt rules, sanitizer behaviour visible to the LLM, or the LLM output schema. Three modes via `EVAL_LLM_MODE`:
  - `replay` (default): cached LLM outputs from `tests/eval/.llm-cache/`, ~2s, deterministic
  - `live`: real LLM calls, saves to cache
  - `refresh`: real LLM calls, overwrites cache

- **Eval tests — extraction goldens** (`tests/eval/extraction/`): the DB-free declarative coverage oracle. Each `<name>/expected.graph.yaml` is scored by ONE shared runner (`extraction.eval.test.ts`) that runs the full in-memory pipeline (`extractEphemeralTopology` + structural config merge) and gates at 90% precision/recall — nodes + edges + symbols, **exact** match, plus negatives. Add coverage by dropping `<name>/fixture/` + `<name>/expected.graph.yaml` (and a `<name>__heldout/` sibling with different names to catch overfitting) — **no per-fixture test code**. **This is the preferred tier for new language / framework / technology coverage** (routes, ORM, brokers, config): declarative, held-out-gated, mostly zero-LLM. Run: `make test-extraction` (or `bun vitest run tests/eval/extraction --config vitest.eval.config.ts`).

- **Eval tests — patterns** (`tests/eval/patterns/`): hand-written multi-file scenarios with **bespoke `it()` assertions** on things a declarative golden can't express — pipeline internals (grounding source/quality, `SymbolRegistry` contents, cross-file resolution decision paths, field required/optional flags) and multi-file structural resolution (Helm/Crossplane). Exercises the full static pipeline (imports, taint, plugin extraction, registry lookups). Typically deterministic (zero LLM calls). Prefer the **extraction** tier above for anything expressible as a graph golden; reach for `patterns/` only when the assertion is about internal state or needs multi-file structural context. Each pattern is a self-contained anonymised micro-repo:
  ```
  tests/eval/patterns/<name>/
    <name>.eval.test.ts              ← test, deterministic by default
    fixture/
      composer.json | package.json   ← language-appropriate manifest
      src/...                        ← anonymised PHP/TS files (acme/inventory only)
    expected.graph.yaml              ← optional, for LLM-replay variants
  ```
  Existing examples: `php-graphql-same-namespace`, `ts-taint-propagation`, `php-psr18-taint-propagation`, `php-symfony-messenger`. Run: `make test-patterns` (~5s, deterministic subset; replay-cached ones run in `make test-eval-golden`).

- **Shared fixtures** (`tests/fixtures/`): anonymised microservice repos used by integration + eval tests. Reuse before creating a new fixture from scratch.

#### When to add an `eval/patterns/` test (mandatory triggers)

Adding or extending a pattern test is **required** when:
- The change touches taint propagation, import resolution, plugin extraction, decorator registries, or any cross-file inference.
- The bug was reported against a real-world codebase (the pattern test is the deterministic, anonymised reproduction — the fix loses this test in CI before the regression reaches production).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coderadius-ai/coderadius](https://github.com/coderadius-ai/coderadius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
