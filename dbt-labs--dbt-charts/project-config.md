---
trigger: always_on
description: Family resolvers (`bar.py`, `line.py`, `area.py`, `scatter.py`, `heatmap.py`, plus
---

# compile/resolve/chart

Family resolvers (`bar.py`, `line.py`, `area.py`, `scatter.py`, `heatmap.py`, plus
`pie.py`/`geo.py`/`simple.py`/`_table.py` for non-cartesian families) turn a
normalized `Chart` into a `Resolved*Chart`. Read `../../AGENTS.md` (compile) and
`../../../AGENTS.md` (core) first; this file covers the cartesian-family
composition invariant.

## Implementation philosophy

Exceeds the usual length budget: the five invariants below (composition, the
family/shared boundary, no inheritance, the axis-cascade context, the zero-anchor
decision) each need a code-anchored citation to stay reviewer-checkable.

### Compose shared policy, never hand-roll it

A cartesian family resolver (bar, line, area, scatter, heatmap) answers a policy
question by calling the shared function that answers it, never by re-deriving the
answer locally. The failure mode this directory is shaped to prevent is a
**skipped** policy call: an absent line is invisible in review, unlike a wrong one.

Every cartesian family calls `plan_cartesian()` / `build_cartesian_axes()`
(`_plan.py`) for the frame and `cartesian_series_naming()` (`_axes.py`) for the
legend-vs-rail decision. A family with "not applicable" to say passes that as an
explicit argument (`scatter`/`heatmap` pass `_NO_RAIL_ENDPOINT_LABELS`; they never
skip the call). A hand-rolled `top_legend` ternary, or reading `primary.legend`
directly instead of letting `cartesian_series_naming()` derive the
author-hid/showed/placed facts from `_authored_legend`, reintroduces the same
invisible gap.

Every family also calls `build_chart_style_context(chart_style_context, normalized)`
itself to get its own `chart_local_style_context` (used for
family-specific style reads like `.bar`/`.line`/`.area` and for `_title_font()`/
`_cartesian_kwargs()`). `plan_cartesian()` takes no chart-local context: it only
ever needs the board-level `chart_style_context`.

`histogram` (`_resolve_histogram` in `bar.py`) is a named exception, not a silent
one: it calls `plan_cartesian()` / `build_cartesian_axes()` for its frame like
every other family, but passes `multiples=None, y=None` and never calls
`cartesian_series_naming()`: a histogram bins x and counts rows, so it authors no
per-row series and has no legend-vs-rail decision to make.
`test_cartesian_family_invariants.py` carries `histogram` in `_FAMILIES` with a
reasoned `pytest.mark.skip` on every invariant that call would otherwise satisfy,
rather than leaving it absent from the parametrization.

### Where the family/shared line sits

Frame and policy are shared: `plan_cartesian()` / `build_cartesian_axes()` run the
identical prelude/postlude (channels, axis cascade bake, both `ResolvedAxisStyle`
builds, tooltip format); `cartesian_series_naming()` owns the
legend/rail decision. Tick, zero and domain math stay per-family: bar's stacked
totals, area's log-domain bake and line's multi-metric zero ladder are genuinely
different math over the same axis, not the same math written five times, and are
unified only by the `_CartesianTickResolution` NamedTuple every family returns, so
the shared postlude can consume any family's result the same way. Do not fold
tick/zero/domain math into the shared prelude/postlude to chase more sharing: a
config flag hiding that divergence trades a visible skipped call for an invisible
default.

### No base class, no template method

The five families are five plain functions (`_resolve_bar`, `_resolve_line`, ...)
dispatched by a `match` in `_dispatch.py`, not subclasses of a common resolver
base. A shared step is a function call at the call site, so a reviewer reads one
file top to bottom and sees whether `_resolve_scatter` calls the same
`cartesian_series_naming()` that `_resolve_line` does, without chasing `super()` or
a hook method up an inheritance chain. No ABC or template method across the
families: it would trade that readability for a symbol-count win the repo doesn't
want.

### The axis-cascade context

`_bake_cartesian_axes()`, called once from `plan_cartesian()`, takes the
board-level `chart_style_context`, never a per-chart one. Chart-local axis patches
are `SkipInheritSlots` fields on `_CartesianChartStyle`, extracted separately as
`AxisOverrides` and re-applied at cascade layers 11-13; passing the chart-local
context into the bake would only double-apply the same patch at a lower priority.
`chart_local_style_context` is still the right input for `_cartesian_kwargs()` and
`_title_font()`: `resolve_title_font()` has no overrides side channel and reads
color/style/decoration/case/line_height straight off the context it is given, so
the board-level context there silently drops a chart-local `style.title.font.color`.

### The zero-anchor decision

Line, area and scatter each call `_bake_y_zero` (`_domain.py`) at resolve, baking
a definitive zero-anchor decision onto `ResolvedAxisStyle.zero_anchored` — the one
fact every family reads off the cascaded y-axis rather than re-deriving. It reads
an author-pinned `scale.continuous.zero` off that same cascaded axis, never the
per-chart/family patch, so a pin authored at board, chart, or family level is
honored identically; absent a pin, the smart-zero heuristic decides. Area's bake

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbt-labs/dbt-charts](https://github.com/dbt-labs/dbt-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
