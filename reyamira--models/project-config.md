---
trigger: always_on
description: Registry-driven multi-source tab: a data-source switcher over 4 sources (AA,
---

# Benchmarks Tab

Registry-driven multi-source tab: a data-source switcher over 4 sources (AA,
Epoch, Arena, LLM Stats), with every view (Detail, H2H, Scatter, Radar, sort,
sidebar) rendered from per-source `MetricDef`s rather than hardcoded field names.

## Files
- `app.rs` — `BenchmarksApp` state, `BenchmarkFocus` (Creators/List/Details/Compare), `BottomView` (H2H/Scatter/Radar), `CreatorGrouping`, `ComparatorMode`, sort/filter types. Key fields: `active_source: usize`, `sort_key: SortKey`, `sort_descending`, `bottom_view`, `comparator: ComparatorMode`, `scatter_x`/`scatter_y`, `radar_group`, `show_sort_picker`/`sort_picker_selected`, `show_glossary`/`glossary_scroll: ScrollOffset`, `detail_scroll: ScrollOffset`. `MultiStore` itself lives on the top-level `App` (`app.multi_store`), not on `BenchmarksApp` — sub-app methods take `&SourceFile` as a parameter.
- `render.rs` — `draw_benchmarks_main()`, `draw_source_bar()`, `build_benchmark_detail_lines()`, `build_glossary_lines()`, `draw_glossary()`, `draw_sort_picker()`, `compare_colors()` (8-color palette)
- `compare.rs` — `draw_h2h_table_generic()`, `draw_scatter()`
- `radar.rs` — `draw_radar()`, `axes_for_group()`, spoke/polygon math (`MAX_AXES = 6`)

## Source switcher
- 1-line source bar above the existing content (`draw_source_bar`): bracketed label per `SOURCES` entry — active = Cyan+BOLD, loaded-inactive = DarkGray, loading = label + `◐` Yellow, failed = label + `✗` Red, then a DarkGray `{ } switch source` hint (mirrors the header's `[/] switch tabs`). Right-aligned for the active source: `updated {relative}` (DarkGray, the data's build-time last-change timestamp from `SourceMeta.fetched_at` — **not** the client fetch time; the app fetches fresh every launch) + ` self-reported` (Yellow) when `verified == false`. Source identity lives here only — no detail-panel attribution line, no status-bar flash on switch.
- `{` / `}` cycle data source prev/next (tab-local; `[` / `]` stay global PrevTab/NextTab). Switching is **state-preserving** (`switch_source` → `reset_for_source`): it keeps `search_query`, the open-weights `source_filter`, `creator_grouping`, and the `reasoning_filter` (reset to `All` only when the target source carries no reasoning metadata, so a stuck invisible filter can't silently empty the list), and keeps a `ReleaseDate`/`Name` sort with its direction (a `Metric(i)` sort is source-specific → falls back to the new source's `default_sort`). Per-source view state still resets: selection/creator indices, all scrolls, `scatter_x`/`scatter_y`, `radar_group`, `bottom_view`, and all popups. Compare selections (`App.selections`) carry over by **exact `ModelRow.id` match, then `benchmarks::normalize_id` match** (the four sources spell the same model differently — `grok-4-3` vs `grok-4.3`, `DeepSeek-V3-1`, `zai-org/GLM-4-7`, dated Arena ids — so exact-only looked randomly flaky), order-preserving (compare-color stability) and deduped (normalization folds e.g. `-thinking` variants onto their base); ids with no counterpart drop, then `update_bottom_view` runs so a compare→browse demotion fires when <2 ids survive.
- `r` — **refresh the active source** (`Message::RefreshBenchmarkSource`). Stale-while-revalidate: the current data keeps rendering while a fresh `fetch_source` runs on tokio, delivered as `Message::DataSourceRefreshed(idx, Option<SourceFile>)`. `Some(file)` → `set_loaded` + the same `finalize_loaded_source` enrichment + a **state-preserving rebuild** (`rebuild_preserving`: keeps sort/search/filters/grouping, remaps `selections` + the focused row by id, falls back a stale `Metric(i)` when the refreshed metric list shrank); status `Refreshed {name}`. `None` (fetch failed) → keep the existing loaded file untouched (never `set_failed`s good data); status `Failed to refresh {name} — keeping current data`. A `Loading`/`Failed` source stays recoverable via `r`.
- Sources load progressively; selecting a still-loading/failed source shows the standard loading/error state.

## Column picker (`C`, browse mode)
- `visible_columns: Vec<usize>` (metric indices, file order, default empty; **persisted** — picker save mirrors the selection into `config.benchmarks.columns` as per-source metric ids and writes config.toml; switch/first-load/refresh re-resolve saved ids via `App::restore_saved_columns`/`apply_saved_columns`, dropping stale ids) + picker state (`show_column_picker`/`column_picker_selected`/`column_picker_pending` — Enter applies the pending set, Esc discards). `C` guarded to browse mode in event.rs; `handle_column_picker_keys` intercepts all keys (incl. `q`) while open.
- `effective_columns()` = visible columns plus the active sort `Metric(i)` appended when absent; ReleaseDate sort keeps the Released column; Name sort adds nothing. Metric columns are 11 wide, headers via `multi::short_label` (curated `short_label` else truncated label), sorted column's header Cyan+BOLD. Width cap keeps the name column ≥ 10 chars and drops excess visible columns from the right — the sort column always survives (resolved from `sort_key`, not by position).

## Glossary popup (`i`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reyamira/models](https://github.com/reyamira/models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
