---
trigger: always_on
description: Guidance for humans and AI coding agents working in this repository.
---

# Contributing conventions — portable-llm-wiki

Guidance for humans and AI coding agents working in this repository.

## Hard rules

- **No emojis, ever.** Do not use emojis or decorative pictographs
  anywhere — not in code, UI strings, button labels, log messages,
  commit messages, comments, or documentation. This includes glyphs
  like check marks, warning signs, clipboards, and similar
  (`✓`, `⚠`, `📋`, etc.). Use plain words instead: render "copied",
  not "copied ✓". Typographic punctuation (em dash `—`, en dash `–`,
  directional arrows in prose) is acceptable; emoji and pictographs
  are not.

## Project layout

- `backend/` — FastAPI app (Python). Tests under `backend/tests/`,
  run with `backend/.venv/bin/python -m pytest`.
- `frontend/` — Next.js app (TypeScript/React). Tests run with
  `npx vitest run`; type-check with `npx tsc --noEmit`.
- `render.yaml` — Render Blueprint for the backend. The hosted service
  is Blueprint-managed with autoDeploy; keep this file in sync with the
  live dashboard so the two never drift.

## Testing

- Run the full relevant suite before claiming work is done:
  backend `pytest`, frontend `vitest run` + `tsc --noEmit`.
- Prefer behavior/invariant assertions over change-detector snapshots.

## Commits

- Conventional commits: `fix:`, `feat:`, `refactor:`, `docs:`,
  `chore:`. Concise subject, body explaining the why.
- Never auto-commit; commit only when explicitly asked. Keep unrelated
  work in separate commits.

<!-- puppetmaster:rules:begin -->
<!-- managed by `puppetmaster install-rules`; delete this whole block to disable -->

# Puppetmaster orchestration

Puppetmaster is an MCP-based agent orchestrator with structured worker
swarms, durable SQLite state, tiered model routing, and zero-token
follow-ups via stored artifacts. When Puppetmaster's MCP server is
registered (`puppetmaster install-cursor-mcp` or
`puppetmaster install-codex-mcp`), the `puppetmaster_*` MCP tools are
available in this environment.

## Trigger convention (must obey)

When the user says **"Use Puppetmaster to …"**, **"PM this …"**, or
otherwise names Puppetmaster for a task, route that work through the
`puppetmaster_*` MCP tools — do not answer inline.

## Delegate-first gate (default path)

Before attempting multi-step work inline, start a Puppetmaster verb
(`puppetmaster_start_cursor_swarm`, `puppetmaster_start_swarm`,
`puppetmaster_start_implement`, or the matching sync verbs) when the
task is any of:

- Multi-file (3+ files) or cross-cutting refactor/migration
- An audit, review, or "find all X" search
- Work whose result will be reused later in this or a future session

Swarms and reviews run read-only analysis; building goes through
implement. Recall prior results with `puppetmaster_artifacts <job_id>`
at zero token cost.

Reach for a Puppetmaster verb **before** native broad search/exploration:
prefer `puppetmaster_codegraph_search` / `_context` over a repo-wide
`Grep`/`Glob`/`find`, and a swarm over the built-in `Task` tool, for any
multi-file investigation. When unsure whether a task qualifies, run the
classifier-backed gate — `puppetmaster_route_task` (or
`puppetmaster should-delegate "<prompt>"`) — which returns a delegate /
inline verdict and a suggested verb with zero LLM cost.

For deterministic enforcement, the user can install host hooks
(`puppetmaster install-hooks`) that inject this directive on prompt submit
and deny-redirect broad native exploration automatically. The kill switch
is `PUPPETMASTER_AUTO_INVOKE_DISABLED=1`.

## Label every job you start (do it by default)

When you start any job verb (`puppetmaster_start_*`, `puppetmaster_edit`,
or the matching sync verbs), pass a short human-readable `label` (3–6
words, e.g. `"auth refactor audit"`). It becomes the job's headline on the
dashboard and in `puppetmaster_jobs`, so runs stay scannable instead of
reading as bare `job_<hash>` ids. Omit it only for throwaway one-off runs;
when absent, Puppetmaster falls back to a title derived from the goal.

## CodeGraph-first exploration (must obey)

CodeGraph is the default way to explore code — graph every directory you
interact with, then explore the graph instead of crawling the tree:

1. **Graph it first.** Before exploring any directory (the workspace root
   or a subtree you're diving into), check `puppetmaster_codegraph_status`;
   if it has no `.codegraph/`, run `puppetmaster_codegraph_init`
   (`index: true`) — it returns immediately and indexes in the background.
   Do not start grepping while you wait.
2. **Ask the graph, not the tree.** Resolve "where is X / what calls Y /
   what implements Z" with `puppetmaster_codegraph_search` /
   `_context` / `_affected` / `_files`, then `Read` only the files it
   points to.
3. **Partial coverage is still coverage.** CodeGraph indexes the languages
   it supports; unsupported files simply don't enter the graph. When part
   of the tree is ungraphable, still answer from the graph for everything
   it covers and scope native search narrowly to the ungraphed paths
   only — never re-crawl directories the graph already covers, and reuse
   that shared context instead of letting multiple workers/agents each
   re-explore the same graphed code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [professorpalmer/portable-llm-wiki](https://github.com/professorpalmer/portable-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
