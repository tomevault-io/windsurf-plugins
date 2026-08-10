---
trigger: always_on
description: See `TRIAGE.md` at the repo root for the priority rubric (P0/P1/P2/P3 + severity labels + milestone scheme). Apply it when classifying new issues or proposing what to work on next. Current milestones live on GitHub: `v1.89.0` (in flight — Template_Version Type picklist fix + CSS 2.1 guidance + #60/#72), `v1.90.0`, `Backlog`.
---

# CLAUDE.md — Portwood

## Triage

See `TRIAGE.md` at the repo root for the priority rubric (P0/P1/P2/P3 + severity labels + milestone scheme). Apply it when classifying new issues or proposing what to work on next. Current milestones live on GitHub: `v1.89.0` (in flight — Template_Version Type picklist fix + CSS 2.1 guidance + #60/#72), `v1.90.0`, `Backlog`.

## Mission

Maintain Portwood — a native Salesforce 2GP package for generating Word and PDF documents from any Salesforce record. Work is roadmap-driven via the GitHub issue board; treat it as the source of truth for what's in flight.

When picking up work, prefer the highest open priority on the smallest milestone. P0 silent-corruption bugs jump the queue. Community-contributed fixes (the `community-contribution` label) are usually fast wins because the reporter has already done the diagnostic work.

## Critical: three merge-tag resolution paths

`DocGenGiantQueryAssembler` does **not** call `processXml()`. A fix to section-tag logic in `processXml` only covers row-level loop bodies. Parent-level tags outside the loop are resolved by `DocGenGiantQueryAssembler.resolveParentMergeTags()`, and grand-total aggregates by `resolveGiantAggregateTags()`. If a parser-level change needs to behave consistently for templates that fall into the giant-query path (>2000 child rows), the logic has to be mirrored in the assembler or routed through `processXmlForTest` the same way format-suffix tags already are. Always check whether your fix needs the same change in the giant-query parent path and add an e2e-07 assertion either way.

## Critical: {#ChartBucket} tag (v1.91+) — 4 resolution paths, 5 modifiers

`{#ChartBucket:relationship:field[:modifier1=value1&modifier2=value2&...]}body{/ChartBucket}` aggregates a child collection by `field`, exposing per-bucket data (`{key}`, `{count}`, `{percent}`, `{max_percent}`, `{color}`, `{index}`, `{key_label}`) inside the body. The bucket list is sorted desc by count, alpha by key for ties.

**Five modifiers** (composable; v1.91 surface):

| Modifier                                | Behavior                                                                                                                                                              |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `colors=#aaa,#bbb,#ccc`                 | Override default 8-color palette, cycles by row index                                                                                                                 |
| `where=Field='Value' AND Other != null` | Sanitized SOQL fragment appended to chart's WHERE. Forces SOQL fallback.                                                                                              |
| `split=;`                               | Multi-select delimiter. Splits combo values per respondent, percentages sum >100% expected.                                                                           |
| `groupBy=Field__c`                      | Cross-tab pivot. Each row gets `cols` sub-list with `{#cols}{key}{count}{percent}{/cols}` body iteration. Synthetic Total column appended last. Forces SOQL fallback. |
| `colSort=val1,val2,...`                 | Author-controlled column ordering (for `groupBy=`). Named values appear first in order; remaining values alpha-sorted; Total always last.                             |

**Four resolution paths** (chart resolver mirrors what merge-tags already do — all four must stay consistent):

1. **In-memory** — `DocGenChartBucketResolver.preprocessInline` against pre-loaded relationship records. Used when child count <2000 AND no `where=`/`groupBy=`. Single SOQL-free path, fastest.
2. **SOQL fallback** — `tryFallbackSoqlAggregateAdvanced` when relationship isn't on the data map OR `where=`/`groupBy=` force it. Schema-auto-discovers child object + FK via `ChildRelationship`. Issues a `GROUP BY` aggregate, constant-cost regardless of row count. **This is how 30K-scale templates work** — Query_Config\_\_c omits the chart-target relationship, retriever doesn't eager-load, chart aggregates server-side.
3. **Parent-level** — `DocGenGiantQueryAssembler.resolveParentMergeTags()` regex skips `{#…}` prefixes, so charts pass through. Then `processXmlForTest` routes through the inline path.
4. **Giant-query parent** — `DocGenGiantQueryAssembler.resolveGiantChartBuckets` for charts targeting the giant relationship in giant-query templates. Same modifiers, same shape.

**SOQL budget**: 50 chart aggregates per transaction (static `DocGenChartBucketResolver.chartSoqlBudget`). When exhausted, charts render a sentinel "Chart limit reached" bucket — never silently empty. Tune in resolver if templates pathologically stack >50 charts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Portwood-Global-Solutions/Portwood](https://github.com/Portwood-Global-Solutions/Portwood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
