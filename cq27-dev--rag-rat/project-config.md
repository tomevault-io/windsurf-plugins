---
trigger: always_on
description: Guidance for coding agents working in the `rag-rat` repository.
---

# AGENTS.md

Guidance for coding agents working in the `rag-rat` repository.
(`CLAUDE.md` is a symlink to this file.)

## Prefer the rag-rat MCP for code browsing

This repo ships `rag-rat` — a local repo-intelligence index and MCP server — and it is indexed by
its own server (dogfooded). **Heavily prefer the `rag-rat` MCP tools over raw `grep`/`cat`/file
reads when browsing or understanding code.** One MCP call returns more context, faster, than a
shell sweep, and it surfaces *drive-by repo memories* (source-anchored invariants, decisions,
risks) attached to the code you're touching — context you would otherwise never see.

Reach for these first:

- **`semantic_search`** — "where is this concept implemented?" Returns current source chunks with
  inline graph (callers/callees), git, and GitHub papertrail, all validated against current source.
- **`symbol_lookup`** — exact/fuzzy symbol resolution (Rust/TS/Kotlin/C/C++/Python/Swift/Go), with
  any bound memories attached.
- **`impact_surface`** — the coding preflight before editing a symbol: graph callers/callees,
  tests, git history, papertrail, and **repo memories** crossing the call path. Run it before
  changing anything non-trivial.
- **`find_callers` / `trace_callees`** — reverse/forward graph traversal instead of grepping for
  call sites.
- **`read_chunk`** — current text for a chunk with anchor validation + graph + memories.
- **`repo_brief` / `repo_clusters`** — orientation (spine, churn, god-modules, ownership clusters).
- **`important_symbols`** — load-bearing symbols by (SCIP-aware) PageRank; pass `personalize` to
  bias toward what you're editing. Compiler-grade once `rag-rat oracle run` has run.

**Symbol handle:** symbol-returning tools emit `id`, an opaque `sym_<hex>` token — the stable handle
to cache and pass back into graph/impact/memory tools as the `id` param (copy verbatim; never parse
it as a number). There is no numeric `symbol_id` on the wire (it's an internal rowid reassigned on
every reindex). Use `ref` (the `path::name` qualified name) for the human-readable identity. The
symbol-tool params are `ref` / `id` / `lang` (formerly `symbol_path` / `logical_symbol_id` /
`language`).

Why this beats grep here:
- Results carry **provenance**: confidence labels, coverage warnings, and raw evidence, so you can
  judge them rather than trust them blindly.
- **Drive-by memories**: a function may carry an `Invariant`/`Decision`/`Risk` memory that explains
  a non-obvious constraint. Grep can't show you that; the MCP tools attach it automatically. See
  *Record durable learnings as rag-rat memories* below — capturing these is part of the job.
- The index is **kept fresh by git hooks** (see below), so what the MCP returns matches HEAD.

Fall back to direct file reads/edits for the actual *writing* of code, and to confirm exact text
before an `Edit`. Use the MCP to *find and understand*; use the file tools to *change*. (The MCP
server is read-only on source — it never edits files; it writes only its own SQLite index.)

## Record durable learnings as rag-rat memories

**Running the gate is required, not optional — producing a record is not.** Before you finish a
task, ask: **could the next agent recover this by reading the repo?** If yes, don't record it. A
memory that restates the code, the types, the tests, or the history leaves the reader *worse off*
than none at all — it costs attention and returns nothing. On most tasks the honest answer is
**record nothing**, and that is the common outcome, not a failure to try hard enough. What passes
the gate is what you had to read three files and reason for ten minutes to learn, and that the repo
states nowhere — don't let that evaporate.

**Rejected alternatives lead.** The approach *not* taken leaves no artifact anywhere — not in the
diff, not in the types, not in the history — which makes it at once the most-asked question about
unfamiliar code and the least-supplied answer. If your work settled a choice, record the alternative
and why it lost (`RejectedAlternative`) before anything about what the code now does.

**Why rag-rat and not your own notes:** rag-rat memories live in this repo's shared index, so they
surface for **every** agent that uses the rag-rat MCP — Claude Code, Codex, and any future tool —
not just the one that wrote them. An agent's private/session memory (e.g. Claude Code's file memory,
Codex's own store) is invisible to the others. rag-rat is the **cross-agent memory layer**; put
anything another agent would benefit from here.

**Translate; never store the trajectory.** What you did, in what order, is worth less than nothing
to the next reader — a replayed trace scores worse than having no memory at all. Distill it to a
situation and an action — *when X, do Y, because Z* — and quote the evidence you generalised from
(the failing line, the error, the constraint). Situated questions are not answered by general
advice, and the quote is what spares the reader from trusting a summary of a summary.

**Revise before you create.** A memory that has drifted actively misleads, while a missing one merely
fails to help — so correcting or retiring a wrong record is the highest-value write available.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cq27-dev/rag-rat](https://github.com/cq27-dev/rag-rat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
