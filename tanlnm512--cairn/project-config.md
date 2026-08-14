---
trigger: always_on
description: This workspace uses a local knowledge graph (cairn) for codebase intelligence.
---

# Codebase Intelligence System

This workspace uses a local knowledge graph (cairn) for codebase intelligence.
All AI coding agents working in this workspace should use these tools.

## MCP Server
- Name: `cairn` (auto-connected at session start)
- Transport: stdio
- 27 tools across 5 layers: graph (9), knowledge base + compass (5), memory (8), knowledge (5)
  (`explore` is the recommended first call -- it aggregates the graph layer;
  `ask_compass` is the cross-layer router)

## Shipping a change — MANDATORY workflow (agent trigger)
TRIGGER: the moment you finish editing and are about to commit, push, or open a
PR — STOP and follow `docs/contribution-workflow.md` end to end:
`branch → pre-commit run --all-files → conventional commit → push feature branch
→ PR (fill the audit checklist) → watch CI → fix on the same branch → post-merge
cairn update + record_memory`.

Hard rules (do not violate):
- Never push directly to `main` (it skips the PR-title/dependency-review gates and the review layer).
- Never `git commit --no-verify` past a pre-commit failure (that defeats Layer 0; only a human may decide to).
- Commit AND PR title must be conventional: `type(optional-scope): subject` (`feat fix chore docs ci refactor perf test build style revert`).
- Do not skip the PR template's audit checklist — it is the Layer 2-3 review gate.

The explore-first / before-editing / after-task sections below are the *how*;
`docs/contribution-workflow.md` is the *procedure* for landing a change. CI
(Layers 0-1) enforces the automatable parts; this workflow covers the rest.

## Workflow: explore-first

### For almost any question -- "how does X work", a flow, surveying an area:
1. Call `explore(query)` FIRST. It returns matching symbols' verbatim source
   grouped by file, the call paths between them (including ambiguous dispatch
   hops), and a blast-radius summary -- one call, one answer.
2. Reach for the specific tools only to drill down when `explore` is thin:
   - `ask_compass(query)` -- cross-layer routing (graph + wiki + compass + memory)
   - `get_callers` / `get_callees` / `impact_analysis` -- deeper call-graph traversal
   - `search_knowledge` / `recall_memory` -- knowledge-layer questions `explore` doesn't cover

### When to escalate beyond explore (one trigger per tool)
`explore` makes three trade-offs by design. Escalate only when you hit a limit:

| explore's limit | You need... | Escalate to |
|-----------------|-------------|-------------|
| Blast radius is depth-2 only | Recursive callers (breaking change) | `impact_analysis(name)` + `cross_repo_deps(repo)` |
| Neighborhood is unordered | Execution order (what runs when) | `trace_flow(entry)` |
| Results are pure L1 structural | Why/decisions/wiki/tribal knowledge | `ask_compass(query)` or `recall_memory(query)` |
| FTS5 seeds are token-based | Meaning-based match (synonyms) | `semantic_search(query)` |

Escalations are additive -- call them *after* `explore` to go deeper, not
instead of it. `explore` already gave you the seed names and file locations
the escalation tools need.

### Before editing a file, ALWAYS:
1. Call `ask_compass(file_path="<path>")` to load compass + memory context
2. Call `find_definition` for any symbol you need to understand
3. Call `get_callers` to understand who depends on what you are changing (within-repo)
4. Call `cross_repo_deps(repo_name)` for cross-repo blast radius
5. Call `impact_analysis(symbol_name)` if making breaking changes (within-repo recursive)

### Resolution-aware querying (precise vs fuzzy)
`get_callers`, `get_callees`, and `impact_analysis` default to **precise**:
they only follow edges the resolver could pin to exactly one definition.

- **Empty precise result ≠ "no callers".** It means "no *resolvable* callers."
  Before concluding a symbol is unused, retry with `fuzzy=True`.
- **Precise is ground truth for blast radius** — not inflated by name collisions.
- **Fuzzy is a candidate list, not truth** — verify each against actual code.
  A fuzzy result for `invoke` can span 200+ sites across repos/languages that
  merely share the name.
- **`resolution` label:** `exact` = trusted; `ambiguous` = multiple candidates,
  resolver declined to guess; `unresolved` = external/stdlib.

When precise is right: impact, refactoring, signature changes.
When fuzzy is right: auditing, dead-code hunting, exploring unfamiliar code.

### When you need architectural context:
- Call `get_compass(module_name)` for a 25-35 line navigation guide
- Call `search_knowledge(query, type_filter="Wiki")` for feature/architecture documentation

### When you need past decisions:
- Call `recall_memory(query)` -- symbol/title-keyed, NOT full-text. Query by
  symbol name or title tokens ("ApiFactory", "backoff"), not natural language.

### After completing a task, ALWAYS:
1. Run `cairn update` to refresh the graph with your changes
2. Call `record_memory` for any learnings:
   - type="decision" for architectural choices made
   - type="pattern" for reusable code patterns discovered
   - type="mistake" for errors others should avoid
   - type="workaround" for non-obvious solutions used
3. Set confidence (0.0-1.0) based on how sure you are

## PR review (the audit gate)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanlnm512/cairn](https://github.com/tanlnm512/cairn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
