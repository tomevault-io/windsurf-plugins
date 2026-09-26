---
trigger: always_on
description: Guidance for Claude Code working in openalgo-charts. This file carries what is **not
---

# CLAUDE.md

Guidance for Claude Code working in openalgo-charts. This file carries what is **not
discoverable by reading the code**: conventions, invariants, and the standard any UI built
against this engine is held to. Structure and commands are discoverable, read them from
the repo.

## What this project is

A from-scratch, dependency-free HTML5 canvas charting engine. Nine lazy ESM tiers (base,
trade, transform, profile, indicators, draw, webgl, workspace, widget), zero runtime dependencies,
enforced Brotli budgets. It is a **general library that ships an Indian default**, not an Indian library:
IST is the default timezone, never an assumption baked into behaviour.

**The engine ships no DOM.** The base and the seven other DOM-free tiers contain no toolbar,
dialog, menu or picker; those live in the host. `openalgo-charts/widget` is the one tier
that does ship them: a packaged host that drives the engine only through its public API,
kept out of every other bundle by the ESLint tier ACL and `npm run shake`. The yfinance
demo (`examples/yfinance/index.html`) is the reference host and the place to prove a
feature is usable, not just present.

## Writing rules

- Keep comparison brands out of source code, tests and comments. Describe behavior
  with generic terms; keep comparative research outside this repository.
- No emoji or icons anywhere: code, comments, log messages, commit messages, docs, tests,
  or terminal output. Plain text labels only.
- No em dashes or en dashes anywhere. Use a comma, colon, parentheses or a full stop. A
  plain hyphen inside a compound word like read-only is fine.
- Comments explain **why**, not what. Match the density and voice of the surrounding file.
- Conventional Commits.

## UI standard for host chrome

**Borrow the craft, not the design.** Professional terminals set the bar for density,
crispness and finish, and that bar is the one to clear. They do not set the layout, the
grouping, or the words. Do not reproduce another product's tab taxonomy, its panel
arrangement, or its label phrasing: openalgo-charts has its own identity and copying
someone else's chrome forfeits it, quite apart from being someone else's work.

Standard domain vocabulary is shared property and should be used plainly: logarithmic,
percent, indexed to 100, precision, timezone, invert. Product-specific phrasings are not,
and neither is a particular way of carving settings into tabs. Where a competitor's label
is the obvious industry term, use it. Where it is their turn of phrase, write our own.

The rest of this section is about craft, and applies whatever the layout ends up being.
Each rule is written down because it was got wrong once:

**Scrollbars.** Never leave a default scrollbar on a dark surface. A white OS scrollbar
against a dark panel is the single most obvious tell that a UI was not finished. Style
`::-webkit-scrollbar` (track, thumb, thumb:hover) and set `scrollbar-color` and
`scrollbar-width: thin` for Firefox. The thumb belongs a step lighter than the panel, not
white, and the track should read as part of the panel.

**Colour controls are small square swatches, not blocks.** A colour input is roughly a
26 to 28 px rounded square. It is NOT a full-width bar: a 140 px colour block is a bug,
not a style choice. `.swatch` already exists at 20 px with a 5 px radius; reuse that
vocabulary rather than inventing a second one.

**Up and down colours share one row.** A property with a bullish and a bearish colour is
one labelled row carrying its checkbox and both swatches side by side:

    [x] Body      [green] [red]
    [x] Borders   [green] [red]
    [x] Wick      [green] [red]

Not a BODY section header followed by separate Up and Down rows. The stacked form triples
the height of every panel and is what forces a scrollbar to appear at all. The settings
schema must therefore be able to express a **paired colour control**, not only single
colours, or the host cannot render this shape.

**Controls are crisp and compact.** Prefer a dense panel that fits without scrolling over
a roomy one that does not. Section headers are small, uppercase and muted. Rows are tight.

**No browser-default form controls on a dark panel.** A native blue checkbox and a native
`<select>` chevron both break the theme. Style checkboxes (dark fill, subtle border, a
clear tick when checked) and selects (panel background, custom chevron, no OS styling).

**Tab lists carry icons.** A settings dialog's left rail pairs each tab with a small
glyph. The demo has an inline SVG icon helper; use it rather than an icon font.

**Dialog furniture.** Title left, close affordance top right, actions bottom right with
the confirming action last, and any secondary control (a template picker) bottom left.

## Shipping a change: every surface that repeats a fact

The same handful of facts (tier count, tier sizes, indicator count, tool count,
chart-type count, test count) is written out by hand in eight places. There is no
single source for them, so a release that updates one and not the rest leaves the
project contradicting itself. That is not hypothetical: the architecture diagram
advertised "under 50 KB" and four tiers for several releases while the base engine

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marketcalls/openalgo-charts](https://github.com/marketcalls/openalgo-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
