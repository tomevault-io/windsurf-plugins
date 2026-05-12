---
trigger: always_on
description: Handbook, framework, and toolkit for agentic programming practices. This repo is a hybrid: part knowledge base (LLM Wiki pattern), part software (pi-agent, tools), part reference library. Mistakes to prevent: writing into `sources/` (immutable), editing wiki pages without updating `wiki/index.md` and `wiki/log.md`, and mixing knowledge-base content with project software.
---

# devstack - Agent Guide

Handbook, framework, and toolkit for agentic programming practices. This repo is a hybrid: part knowledge base (LLM Wiki pattern), part software (pi-agent, tools), part reference library. Mistakes to prevent: writing into `sources/` (immutable), editing wiki pages without updating `wiki/index.md` and `wiki/log.md`, and mixing knowledge-base content with project software.

## Non-Negotiables

1. **Commit after every logical work unit.** Do not wait to be asked. A logical unit is any self-contained piece of completed work: a wiki ingest, a new file, an edit to docs, a config change, scaffolding, a bug fix. If it's done, commit it. Multiple small commits are always better than one big commit at the end.
2. **Never use `git add .`, `git add -A`, or `git commit -a`.** Stage files explicitly by name.
3. **`sources/` is immutable.** Never modify a file after it has been filed there.

## Summary

- Primary purpose: personal agentic practices KB + supporting software
- `AGENTS.md` is the canonical instructions file; `CLAUDE.md` is symlinked to it
- Source-of-truth docs: `WORKLOG.md` (session history), `README.md` (repo structure), `docs/WIKI.md` (wiki schema), `wiki/index.md` (wiki catalog), `wiki/log.md` (wiki operations log)
- The wiki schema (ingest/query/lint operations, page conventions) lives in `docs/WIKI.md`

## Symlink Convention

All repos use `AGENTS.md` as the single instruction source. `CLAUDE.md` is a symlink:

```bash
# In any repo root:
ln -s AGENTS.md CLAUDE.md
```

This ensures Claude Code and Codex read the same file. The symlink should be committed to git. When editing instructions, always edit `AGENTS.md` directly.

## Project Overview

This repo has two distinct layers:

**Knowledge base** — the LLM Wiki pattern (Karpathy). Sources go into `sources/`, the agent compiles synthesized pages into `wiki/`, and `inbox/` is the staging area for unprocessed material. The agent owns `wiki/`; humans own `sources/` and `inbox/`.

**Software and tools** — pi-agent, RTK configs, scripts, and other software live in `projects/` and `tools/`. These follow normal development workflows, not wiki operations.

## Key Files

| Path | Purpose |
| --- | --- |
| `README.md` | Repo overview, directory roles, setup |
| `AGENTS.md` | This file — agent instructions |
| `WORKLOG.md` | Append-only session log — what was done, decisions, next steps |
| `docs/WIKI.md` | Wiki schema — operations, page conventions, filing rules |
| `wiki/index.md` | Agent-maintained catalog of all wiki pages |
| `wiki/log.md` | Append-only chronological operations log |
| `docs/` | Working project docs for this repo (dev notes, research) |
| `writing/` | Your authored content — writeups, talks, slides |

## Directory Ownership

| Directory | Owner | Mutability |
| --- | --- | --- |
| `inbox/` | Human drops, agent processes | Transient — items move to `sources/` after ingest |
| `sources/` | Human curates | Immutable once filed — agent reads, never modifies |
| `wiki/` | Agent | Agent creates, updates, cross-links; human reads |
| `writing/` | Human | Your authored work — not for agent ingest |
| `docs/` | Human or agent | Project working docs, not wiki content |
| `projects/` | Human or agent | Software — normal dev workflow |
| `tools/` | Human or agent | Scripts and configs — normal dev workflow |

## Workflow Expectations

### Before Starting

- Read `WORKLOG.md` — check the latest entry for context on recent work and next steps
- Check `git status -sb`
- For wiki operations, read `wiki/index.md` to understand current wiki state

### Wiki Operations

Full wiki schema is in [`docs/WIKI.md`](docs/WIKI.md) — operations, page conventions, index/log formats, subdirectories, filing rules.

- **Ingest**: process `inbox/` items → write/update `wiki/` pages → update `wiki/index.md` → prepend to `wiki/log.md` → move originals to `sources/`
- **Query**: read `wiki/index.md` → read relevant pages → synthesize → optionally file answer as new wiki page
- **Lint**: check for orphans, stale claims, missing pages, contradictions, broken `[[wikilinks]]`

Key constraints:
- Every wiki page needs frontmatter (`title`, `tags`, `sources`, `links`) — see `docs/WIKI.md` for format
- Every new or updated page must be reflected in `wiki/index.md`
- Every operation must be logged in `wiki/log.md` (reverse-chronological)
- File sources into the correct `sources/` subdirectory by format/origin — see `docs/WIKI.md` filing rules

### Research and Claim Hygiene

When ingesting research sources or writing wiki pages that reference external work:

- Separate **author claims** from **our verification**. If we haven't reproduced or validated something, say so explicitly.
- Don't invent implementation details. Prefer "unknown/TBD" with links to upstream evidence over plausible-sounding guesses.
- When quoting benchmarks or metrics, include context: hardware, dataset, date, methodology — whatever is needed to interpret the number.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhl/devstack](https://github.com/lhl/devstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
