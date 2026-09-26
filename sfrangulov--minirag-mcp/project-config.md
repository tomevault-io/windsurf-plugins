---
trigger: always_on
description: This file provides instructions and context for AI coding agents working on this project.
---

# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this project.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:7510c1e2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->


## Build & Test

Everything runs through `uv`. There is no other supported invocation.

```bash
uv sync                                   # install, including dev dependencies
uv run pytest -q                          # fast suite; the gate for every change
uv run pytest -m slow -q                  # real embedding model, ~220 MB download
uv run ruff check src tests               # lint
uv run ruff format --check src tests      # formatting (src and tests only, see below)
uv run minirag-mcp status --base-dir .    # smoke the CLI
uv build                                  # sdist + wheel into dist/
```

`ruff format --check .` fails on `docs/superpowers/plans/*.md` — those hold Python
in fenced blocks and are frozen historical records. Scope formatting to `src tests`,
which is what CI does.

The slow test downloads the model on first run and caches it; it is excluded by
default via `addopts = "-m 'not slow'"`. Do not remove that marker to "make CI
faster" — the point is that ordinary runs stay offline.

`uv run --extra ocr ...` leaves the extra installed in `.venv`, and a later plain
`uv run` does not remove it. "The suite is green without the extra" is therefore
unverified until `uv sync --exact` has run — that is the command that takes the
extra back out.

## Architecture Overview

An MCP server and a CLI over one index. Both are thin shells around the same core
and hold no business logic of their own — a behavior change belongs in the core, and
anything user-visible must appear identically through both.

```
src/minirag_mcp/
  server.py      FastMCP app, 11 tools          cli/          9 subcommands
                          \                    /
                           pipeline / store / embedder        <- the core
  ingest/parser.py    markitdown: file, HTML, URL -> Markdown + title
  ingest/pipeline.py  parse -> chunk -> embed -> store (replace by source)
  ingest/scanner.py   recursive walk, extension whitelist, sha256 sync diff
  chunker/            structural split (code fences atomic) then semantic merge
  embedder.py         fastembed, lazy model load, unit-normalized vectors
  store.py            LanceDB: vectors + BM25 FTS over text and title, hybrid search
  scope.py            the single definition of "is this path inside that prefix"
  security.py         document-root containment, URL host rules
  sync.py             one background job, per-file failures never abort it
```

Retrieval is hybrid: a vector query and an FTS query run separately and are fused by
weighted Reciprocal Rank Fusion in `store.search`. `RAG_HYBRID_WEIGHT` is the keyword
side's share. LanceDB's own `LinearCombinationReranker` was tried and rejected — it
mixes `1 - L2_distance` with raw BM25, whose scales are incomparable, so an exact-term
hit can never outrank a strong vector match at any weight. Do not reintroduce it.

## Conventions & Patterns

- **English everywhere** — code, identifiers, comments, docs, commit messages.
- Comments state constraints the code cannot show. Never narrate what the next line
  does, and never explain why a change is correct — that is a note to a reviewer that
  becomes noise the moment the PR merges.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfrangulov/minirag-mcp](https://github.com/sfrangulov/minirag-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
