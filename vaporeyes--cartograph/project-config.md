---
trigger: always_on
description: AI-assisted code context engine. Three-phase pipeline: map (index files), lens (select context), run (assemble + generate).
---

# cartograph Development Guidelines

## Project Overview

AI-assisted code context engine. Three-phase pipeline: map (index files), lens (select context), run (assemble + generate).

## Tech Stack

- Python 3.12+, Click (CLI), litellm (model calls), Pydantic (schemas)
- asyncio + aiohttp (concurrent I/O), xxhash (content hashing), tiktoken (token counting)
- pathspec (.gitignore parsing), PyYAML (config)

## Project Structure

```text
src/cartograph/          -- Main package (src layout)
  cli.py                 -- Click command group
  config.py              -- Three-layer config loading
  mapper/                -- Phase 1: code map generation
  lens/                  -- Phase 2: context selection
  assembler/             -- Phase 3: prompt assembly + budget
  docs/                  -- Auto-docs generation from code maps
  deps/                  -- Dependency analysis (no LLM, mechanical)
  dead/                  -- Dead code detection via AST (no LLM)
  style/                 -- Style consistency detection (LLM-assisted)
  search/                -- Semantic search via embeddings (numpy + litellm)
  experiment/            -- Automated experiment loop (propose, test, keep/discard)
  review/                -- Diff-aware code review via LLM
  lat/                   -- lat.md knowledge graph integration (parse, scan, context, bootstrap, sync)
  workspace/             -- Cross-repo workspace orchestration
  models/                -- litellm client + cost tracking
  cache/                 -- Content-addressed cache + index
tests/                   -- pytest test suite
  fixtures/              -- tiny-repo, mixed-repo, workspace, experiment-repo, lat-repo test data
  test_mapper.py         -- Scanner, schema, prompt tests
  test_lens.py           -- LensResult, query hash, selector tests
  test_assembler.py      -- Budget, context assembly tests
  test_cache.py          -- Cache store + index tests
  test_config.py         -- Config loading tests
  test_cli.py            -- CLI command tests (init, map, lens, run, status, docs, onboard, deps, dead, style, search, experiment, review, lat, workspace)
  test_experiment.py     -- Experiment loop: schema, git helpers, proposer, single cycle tests
  test_review.py         -- Review: diff parsing, source detection, schema tests
  test_dead.py           -- Dead code extractor, analyzer, confidence, renderer tests
  test_style.py          -- Style sampling, extraction, detection, diffing, renderer tests
  test_search.py         -- Searcher, fallback, index management tests
  test_lat.py            -- lat.md parser, scanner, graph, context, bootstrap, sync, review/experiment integration tests
  test_workspace.py      -- Workspace config, resolver, unified index, cross-repo search/lens tests
  test_deps.py           -- Dependency analyzer + renderer tests
  test_docs.py           -- Docs index, scope hashing, generation tests
  test_models.py         -- Cost tracking + tier selection tests
  test_integration.py    -- Full pipeline tests with mocked models
```

## Commands

```bash
uv run pytest                    # run tests (416 tests)
uv run ruff check src/ tests/   # lint
uv run ruff check --fix src/ tests/  # auto-fix lint
```

## CLI Contract

- All commands output JSON to stdout, progress/errors to stderr
- Exit codes: 0 success, 1 no results, 2 not initialized, 3 budget too small
- `cartograph run` streams raw model output (not JSON) to stdout
- `cartograph onboard --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph deps --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph dead --format mermaid` outputs raw Mermaid (not JSON) to stdout
- `cartograph dead` exit code 1 = no dead code found (clean bill of health)
- `cartograph style` outputs JSON to stdout; exit 0 = outliers found, 1 = no outliers/patterns
- `cartograph style --format markdown` outputs raw Markdown (not JSON) to stdout
- `cartograph search` outputs JSON to stdout; exit 0 = results found, 1 = no results
- `cartograph search --rebuild` rebuilds embedding index without querying
- `cartograph experiment` outputs JSON to stdout; exit 0 = experiments completed, 1 = no experiments, 2 = not initialized, 3 = pre-flight test failed
- `cartograph experiment --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph experiment` requires `--test-cmd` flag (no default test command)
- `cartograph review` outputs JSON to stdout; `--stdout` outputs Markdown
- `cartograph lat init` generates mechanical lat.md/ from code maps; exit 0 = created, 2 = not initialized, 4 = already exists
- `cartograph lat generate` uses LLM to produce concept-oriented lat.md/ with design rationale and cross-references
- `cartograph lat init --force` / `lat generate --force` additive merge (only adds new files, preserves existing)
- `cartograph lat sync` detects stale sections, orphaned refs, uncovered modules; exit 0 = issues found, 1 = in sync, 2 = not initialized
- `cartograph lat sync --apply` writes proposed changes to lat.md/
- `cartograph lat export --vault PATH` exports lat.md/ to Obsidian vault with converted links, frontmatter, tags, and MOC index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vaporeyes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
