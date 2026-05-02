---
trigger: always_on
description: Operating notes for AI agents (Claude, Codex, etc.) working on splashboard. Humans reading this are welcome too.
---

# AGENTS.md

Operating notes for AI agents (Claude, Codex, etc.) working on splashboard. Humans reading this are welcome too.

## What splashboard is

A customizable terminal splash rendered on shell startup and on `cd`. One-line install into your shell rc, TOML config, fast cached first-paint with background refresh.

**Killer feature**: per-directory `.splashboard/config.toml` (walk-up discovery). Different repos get different splashes automatically. Competitor products (neofetch, fastfetch, starship) don't do this.

**Positioning contexts** — all three are first-class:

| context | value axis | config source |
|---|---|---|
| self / home | daily delight, ambient info | `$HOME/.splashboard/config.toml` |
| self / project | operational (CI, branch, PRs) | `./.splashboard/config.toml` (per-dir, walk-up) |
| other / project | craft + wow for cloners | `./.splashboard/config.toml` shipped with the repo |

## Core architecture: Shape × Fetcher × Renderer

A **widget** is the composition of three independent pieces:

```
Widget = Fetcher × Renderer × Layout slot
```

Splitting those axes is the whole design. Treat them as separate concerns; resist coupling.

### Shape (`src/render/mod.rs::Shape`, `src/payload.rs::Body`)

The **data-shape contract** between fetchers and renderers. Each `Body` variant corresponds to one `Shape`:

- `Text` — exactly one string (clock time, branch name, greeting)
- `TextBlock` — zero or more lines of text (recent commits, welcome notes, todo items)
- `Entries` — key/value rows with optional status
- `Ratio` — a single `0..=1` value + optional label
- `NumberSeries` — `Vec<u64>`, histograms / sparklines
- `PointSeries` — `Vec<(f64, f64)>` in one or more series
- `Bars` — labeled bars
- `Image` — path to PNG/JPEG
- `Calendar` — year + month + optional highlighted day + event days
- `Heatmap` — 2D intensity grid with optional thresholds and edge labels
- `Badge` — short status pill (label + tone), consumed by the `status_badge` renderer
- `Timeline` — chronological entries (commits, releases, notifications)

Adding a new shape means:
1. A new `Body` variant + its `*Data` struct in `payload.rs` (serde-serializable).
2. A new `Shape` enum variant in `render/mod.rs`.
3. An entry in `shape_of()`.
4. An entry in `default_renderer_for()`.
5. At least one renderer that lists the new shape in its `accepts()`.

Shapes are the **only** coupling between fetchers and renderers. If you find yourself thinking "my renderer needs the raw fetcher data", add a new shape instead.

### Fetcher (`src/fetcher/`)

Produces a `Payload`. Two flavors:

- **`Fetcher` (cached, async)** — disk cache with TTL, daemon refreshes in background, renderer reads from cache. Right for anything that does I/O: git2, HTTP, filesystem scans.
- **`RealtimeFetcher` (sync, per-frame)** — recomputed on every draw tick, no cache at all. Right for "right now" values: `clock`, `system_cpu`, `system_uptime`, `clock_countdown`, `pomodoro`. Contract: < 1ms, infallible, no I/O. If you want to put `reqwest` in a `RealtimeFetcher`, it's not realtime — it's cached.

Key invariants:

- Each fetcher declares its supported shapes via **`fn shapes(&self) -> &[Shape]`**. Multi-shape fetchers (`clock`, `basic_read_store`) branch on `ctx.shape` inside `fetch` / `compute`; the runtime validates the config-requested shape against the list before dispatch and renders a placeholder on mismatch instead of crashing. Single-shape fetchers just return a one-element slice.
- Each fetcher declares a **`Safety`** class:
  - `Safe` — renders even in untrusted local configs. Local-only reads, or fixed-host authenticated network (the token only leaves to a known host).
  - `Network` — trust-gated when the URL or query is config-provided (rss, calendar, any fetcher that takes a user URL).
  - `Exec` — **no longer supported**. Plugin protocol (#5) and command widget (#20) are closed. Don't reintroduce.

- `basic_read_store` (`src/fetcher/read_store.rs`) is the escape hatch for "I want a custom widget": user writes `$HOME/.splashboard/store/<id>.<ext>`, splashboard deserializes per the declared shape. Always `Safe` (fixed path, no traversal).

- Fetchers declare their output shape(s) explicitly via `shapes()`. Renderers are compat-checked against the emitted `Body` variant at dispatch time.

- Both kinds register into the shared `Registry` via `with_builtins()`. Lookup is name-keyed; realtime and cached live in the same namespace, same name = collision (last one wins).

### Renderer (`src/render/`)

Consumes a `Payload` + `RenderOptions` and draws into a ratatui `Frame`. Each renderer declares:

- `name()` — used in config (`render = "grid_heatmap"` or `render = { type = "grid_heatmap", align = "center" }`). Names follow a `family_variant` convention (`text_plain`, `text_ascii`, `gauge_circle`, `gauge_line`, `chart_sparkline`, `grid_table`, `list_timeline`, …) so siblings sort together in the catalog.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unhappychoice/splashboard](https://github.com/unhappychoice/splashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
