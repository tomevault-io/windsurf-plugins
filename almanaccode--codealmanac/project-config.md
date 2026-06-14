---
trigger: always_on
description: codealmanac is a living wiki for codebases, maintained by AI coding agents. It documents what the code can't say — decisions, flows, invariants, incidents, gotchas — as atomic, interlinked markdown pages living at `.almanac/` in each repo. Primary consumer is the AI coding agent; humans benefit secondarily.
---

codealmanac is a living wiki for codebases, maintained by AI coding agents. It documents what the code can't say — decisions, flows, invariants, incidents, gotchas — as atomic, interlinked markdown pages living at `.almanac/` in each repo. Primary consumer is the AI coding agent; humans benefit secondarily.

**Full spec:** `/Users/rohan/Desktop/Projects/openalmanac/docs/ideas/codebase-wiki.md` — source of truth. Read it before making design changes. This file is the working context for implementation.

## Design philosophy

Intelligence lives in prompts, not pipelines. When judgment is needed — deciding what a session produced, scoring notability, evaluating a proposed page against the graph — we hand a concrete-but-open prompt to an agent. We do not wrap agents in propose/review/apply state machines, intermediate proposal files, or `--dry-run` rehearsals. The writer owns outcomes and calls the reviewer as a subagent when it wants feedback; there is no orchestration JSON schema between them. Everything is **local-only** (`.almanac/` per repo, `~/.almanac/registry.json` globally, no hosted service), the `.almanac/` namespace is **flat** (no `.almanac/wiki/` subdir — future features get peer files), and **the CLI never touches AI except `capture` and `bootstrap`.** Everything else is pure query and organization over a SQLite index.

## Engineering taste

There is no prize for preserving awkward code. Prefer the structure a new maintainer would understand immediately.

- Prefer obvious architecture over local patching when a bug exposes mixed responsibilities.
- Before accepting a solution, ask whether it creates a one-off mechanism where the existing general model could be extended instead. If it does, require an explicit architectural justification: why the general path is insufficient, whether the exception is temporary or permanent, how it is bounded, and what would make it safe to remove or generalize later.
- Existing special conditions are not automatically legitimate just because they are already in the codebase. This project has been built with AI, and AI agents can leave behind locally effective one-off fixes that were never consciously accepted as architecture. Treat extra flags, copied files, fallback paths, bespoke state, provider-specific branches, and helper scripts as provisional until they earn their place.
- Weight special cases by cost and evidence. Do not remove them reflexively; compatibility, migration, safety, platform, and provider constraints can be real. But a narrow exception with high maintenance cost needs stronger justification than a small localized shim.
- Assume the agents using this repo are capable: they can read files, inspect history, follow wiki pages, call tools, and reason over context. Do not build rigid preprocessing, copied context bundles, artificial staging files, or elaborate orchestration solely because an agent might need help. Prefer giving the agent real source material and a clear contract unless there is evidence that the general agentic workflow fails.
- This is an open-source project, so every new tracked file is public surface area and future maintenance burden. Before adding a file, ask who will own it, whether it belongs in an existing prompt/doc/module, and what keeps it from becoming stale.
- Keep modules honest: a file named `auth.ts` should not secretly mean "Claude auth"; a central status file should not know provider-specific details.
- Short files are good, but responsibility boundaries matter more than line count. Split when a file has multiple reasons to change.
- Delete dead compatibility layers once callers have moved. Compatibility shims are temporary bridges, not architecture.
- When code feels ugly, treat that as design feedback. Naming, file shape, and import direction are part of correctness because they teach future agents how to extend the system.



## Codebase map

| Directory | What it is | Key files |
|-----------|-----------|-----------|
| `bin/` | npm bin shim — error-formatter around `src/cli.ts` | `codealmanac.ts` |
| `src/` | TypeScript source | `cli.ts` (commander wiring), `paths.ts` (walk-up to nearest `.almanac/`), `slug.ts` (kebab-case canonicalization) |
| `src/commands/` | One file per CLI command | `init.ts`, `list.ts`, `search.ts`, `show.ts`, `path.ts`, `info.ts`, `reindex.ts` |
| `src/agent/` | Agent facade, provider registry, provider adapters, prompt loading | `sdk.ts`, `types.ts`, `providers/` |
| `src/indexer/` | SQLite indexer — schema, frontmatter parse, `[[...]]` classifier, freshness | `schema.ts`, `index.ts`, `frontmatter.ts`, `wikilinks.ts`, `paths.ts` (normalization), `resolve-wiki.ts`, `duration.ts` |
| `src/registry/` | Global registry at `~/.almanac/registry.json` — atomic read/write + auto-register | `index.ts`, `autoregister.ts` |
| `src/topics/` | Topic DAG serialized to `.almanac/topics.yaml` + page frontmatter rewrites (slice 3) | `yaml.ts`, `frontmatter-rewrite.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlmanacCode/codealmanac](https://github.com/AlmanacCode/codealmanac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
