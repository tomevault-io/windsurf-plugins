---
trigger: always_on
description: > ↑ [packages/mewbo_graph/CLAUDE.md](../../../../CLAUDE.md) · [root](../../../../../../CLAUDE.md)
---

> ↑ [packages/mewbo_graph/CLAUDE.md](../../../../CLAUDE.md) · [root](../../../../../../CLAUDE.md)

# scg built-in plugin — SCG map + search tools

Scope: `packages/mewbo_graph/src/mewbo_graph/plugins/scg/`. The
deterministic SCG logic does **not** live here — these SessionTools
(`scg_introspect_source`, `scg_build_structure`, `scg_link_entities`,
`scg_finalize_map`, `scg_route`, `scg_observe`, `scg_memory`, `scg_results`,
`agentic_search`) are **thin wrappers** over the SCG core, which now lives
**down** in the same library at `mewbo_graph.scg` (same package, imported DOWN —
no longer a one-way boundary UP into an app).

**`agentic_search` — the high-level self-facing search verb (vs the low-level
graph tools).** Where `scg_route`/`scg_observe` let a task agent inspect
reachability directly, `agentic_search` runs a WHOLE `scg-search` session and
hands back a cited answer. It is **async-by-handle**: `query` starts a run and
returns a `run_id` + `status:"processing"` IMMEDIATELY (a search runs for
minutes — never block the caller's loop); re-call with `run_id` to fetch the
cited answer + `computed_at`; an identical recent query is idempotently reused.
The tool owns NO orchestration (the no-parallel-loop invariant holds) — it drives
the run through the down-only `mewbo_graph.scg.search_launcher.SearchLauncher`
seam the API registers (`RunStoreSearchLauncher`, reusing `SearchRun.start` + the
run store), degrading to a structured "unavailable" error when none is wired.
The id is `search`-classified in core `_infer_operation` → default-allowed, so it
surfaces to any ordinary task agent the `scg` capability grants (#84) — the same
async run/poll shape the external MCP `search`/`get_search_run` tools expose. `scg_results` (#95/#102) is the thinnest of all —
**transcript-as-transport**: it only VALIDATES the search-result entries
(≤50, `extra="forbid"`, relevance/confidence 0..1) and returns `{ok, count}`;
it writes nothing (no store, no sink — the api projects the validated
`tool_input` from the session transcript onto the run event log; a child loop
inherits the parent's event_logger, so a PROBE's emit rides the same
transcript stamped with its `agent_id`). EVERY search agent emits once (#102):
each probe right before its evidence block (its cards, ids salted by its
agent_id api-side), the ROOT before the synthesis for inline-grounded hits
only. Emitting is never terminal — the probe's terminal stays the stop-summary
evidence block. Read the api-side subsystem `CLAUDE.md`
(`apps/mewbo_api/.../agentic_search/scg/`) and Gitea #19 for the durable
architecture decisions.

**`ScgResultEntry.meta` + `ScgResultsArgs.related_questions` (the wire contract
that made connectors' rich returns survive).** A card now carries a structured
fact sidecar `meta: dict[str, str|int|float|bool]` (validators: ≤12 keys, key
≤40 chars, str value ≤200 chars, scalar values only) so every
QUANTITATIVE/ENUMERABLE fact (repo stars/forks/language, package
version/downloads, paper year/citations, issue state/comments) rides as data
instead of being flattened into snippet prose — the playbooks teach the per-kind
vocab and keep `snippet` purely descriptive. **`meta` IS the card's "card_meta"
footer (#111): OPEN-vocab — the agent PROPOSES whatever facts make a hit read
richer, naming the connector's own fields; the console's `resultMeta` classifier
renders any key (counts→`46.2k`, byte `size`→`24 KB`, dates→relative,
`state`/`status`→a colour-coded badge). Do NOT add a second `card_meta` field or
per-kind tool args — the open dict + a generic FE renderer is the whole
mechanism (no over-engineered LLM behaviour).** `related_questions: list[str]`
(≤5, each ≤140 chars) is RUN-LEVEL (alongside `results`): the structured home
for follow-up suggestions, so the synthesis answer drops conversational "If you
want, I can…" offers. Both validate-and-REJECT (no silent truncation — the tool
is validate-only and the model retries on a `ValidationError`). The reason
nothing rode before: `extra="forbid"` dropped any field the schema didn't
declare. The root must NOT re-emit a hit a probe already emitted (probe cards
are already projected — a root duplicate is degraded); it emits `results` only
for evidence it grounded itself, and `related_questions` either way.

**`err_result` now records a FAILED step (core seam).** An scg tool that catches
an internal exception and RETURNS `err_result(code, message)` (the
`str({"error": {"code", "message"}})` envelope — shared with the wiki
`_err_result`) used to record as a SUCCESSFUL tool result, so the loop's
failure machinery never engaged (an embedding-429 rendered as "✓ ok" and the
per-step failure nudge never fired). Core's `tool_use_loop` now detects that
envelope shape on any session-tool return (`_session_tool_error_envelope`, a
pure structural check — core never imports the graph layer) and emits the
`tool_result` event with `success: False` + the envelope message as `error`,
while the model STILL receives the envelope JSON as the tool output. So an
enveloped failure is honestly a failure to the engine without losing the
structured detail. Nothing in the scg tools changed — `err_result` is the
contract; keep returning it for handled failures.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bearlike/Assistant](https://github.com/bearlike/Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
