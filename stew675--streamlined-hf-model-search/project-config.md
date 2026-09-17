---
trigger: always_on
description: Single-file, zero-dependency HTML/JS app that explores HuggingFace base models in a 4-level hierarchy (Author → Base Model → Quant Author → Quant). All logic lives in `streamlined-hf-model-search.html`.
---

# AGENTS.md — Streamlined HF Model Search

## Project Overview

Single-file, zero-dependency HTML/JS app that explores HuggingFace base models in a 4-level hierarchy (Author → Base Model → Quant Author → Quant). All logic lives in `streamlined-hf-model-search.html`.

## Editing AGENTS.md

When modifying this file, write the new content to a temporary file (e.g. `AGENTS.md.new`) and only copy it back to `AGENTS.md` when all edits are done. This minimizes LLM context churn from many small sequential edits against the same file.

## Documentation Files

Read `DESIGN.md` before making changes that touch: popup behavior, CONFIG values, queue/rate-limiting logic, generation guards, render pipeline, data flow, state management, or filtering/injection logic. It contains architecture decisions and design rationale that explain *why* things work the way they do — preventing you from "fixing" intentional design choices (e.g., rejecting `queueMicrotask` for the queue scheduler, keeping L2/L3/L4 render functions separate). When your change introduces a new design decision or resolves a code review finding, append an entry to `CHANGELOG.md` rather than adding inline comments to the source.

## Conventions

- **No external dependencies** — everything inline.
- **Indent**: 2 spaces.
- **Event delegation**: One listener per container after `innerHTML` injection (`_delegatedL2`/`_delegatedL3`/`_delegatedL4` flag); state stored in `_lXState` on the container. Toggle `<button type="button">` for native keyboard Enter/Space handling.
- **ID scheme**: `t{level}-{idx}` (toggles), `d{level}-{idx}` (detail rows), `i{level}-{idx}` (inner containers).
- **Level discrimination**: `<th>` elements carry `data-level="2|3|4"` so sort handlers reject events from nested levels even after `innerHTML` detaches the target from the DOM.
- **Generation guard**: All async functions that mutate shared state capture `const gen = _fetchGeneration` at entry and check `if (isStale(gen)) return;` before any side-effect. The queue manager checks generation both at dequeue time and post-fetch. See DESIGN.md "Architecture Decisions" for rationale.
- **CSS.escape**: Any query selector interpolating user-controlled strings (author names, model IDs) must use `CSS.escape()` to prevent broken queries or injection.
- **Two-tier rendering**: `UI` object handles progressive feedback (status bar, cell badges) via `setStatus`/`queueUpdate` — never touches table structure. `RenderCoordinator` (`RC`) handles structural renders (table rebuilds) via `requestRender`/`_doFullRender` — synchronous only, guarded by `_isRendering`. Async deepening runs in a separate `_asyncDeepenPass` pass after the sync render completes.
- **No structural renders from resolution paths**: `tryResolveModelParam`, `deepenBatch`, and the inline derive loop update state/caches only, queue progressive badge updates via `UI.queueUpdate()`, and schedule structural renders via `_schedulePostDeepenRender` only when filter boundaries are crossed.
- **`refreshAllExpanded(force, allowAsync = true, authorFilter = null)`**: The `allowAsync` parameter distinguishes structural-pass re-renders (`false`, no deepening) from user-triggered refreshes (`true`, deepens via `refreshAuthorL2Section`). When `authorFilter` is set, only that author's subtree is re-expanded via O(1) Map lookup, avoiding a full scan.

## Testing

Open in browser, validate:
1. L1 loads with authors and counts after "Get Results"
2. From/To filter bars change activated pipeline tags
3. Author → L2, base model → L3, quant author → L4 all expand correctly
4. Quant filter chips update all expanded sections
5. Column headers toggle sort direction
6. Links open in new tabs
7. Date/param sliders re-render L1 + all open L2 sections
8. Unknown params in L2 Params column fetch in batches of 4 with loading indicator
9. API call counter ≤4 req/s (1 call per 250ms window)
10. L4 sort by Model ID doesn't collapse L4 content
11. Missing B/M suffix models (e.g. `Qwen/Qwen3-Coder-Next-GGUF`) inherit parent params after deepening
12. Same-author quants appear at L2 under their author (not silently suppressed)
13. Rapid double-clicks don't produce stale renders or duplicate API calls
14. Hidden Models Preview popups: appear at L2/L4 on hover, center over trigger with boundary clamping, stay visible when moving mouse into popup, links clickable
15. Popup hidden count matches L2/L4 trigger count exactly; L2 popup shows all hidden models (no sample cap)
16. Popup sortable columns: click any header to sort, ▲/▼ arrows indicate sort direction
17. "Hide Missing Params" chip hides param-less models from L1/L2
18. API counter flashes amber on 3+ consecutive 429s
19. Clear Cache clears everything, preserves filter/slider state
20. `_fetchSeen` is reinitialized each generation and deduplicates model IDs during ingestion
21. Progressive rendering: status line shows `Fetching models… (N/M)` as each request completes; L1 updates incrementally during fetch
22. Param badge updates progressively as deepening resolves each model (no full table re-render per resolution)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stew675/streamlined-hf-model-search](https://github.com/stew675/streamlined-hf-model-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
