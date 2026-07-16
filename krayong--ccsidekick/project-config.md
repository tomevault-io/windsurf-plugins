---
trigger: always_on
description: Guidance for working on ccsidekick: a Claude Code status line with an animated, reactive character
---

# CLAUDE.md

Guidance for working on ccsidekick: a Claude Code status line with an animated, reactive character
and a full widget layer. No Claude API, no token spend. Network use is limited to non-LLM lookups (
an
optional weekly currency-rate refresh and an optional account-usage call, both off by default), off
the hot render path and enabled only when you turn them on.

## Architecture

A Bun workspace. The workspace root is `ccsidekick-workspace` (private). The engine is
`packages/core` (published as `ccsidekick`); each character is `packages/packs/<name>` (published as
`@ccsidekick/pack-<name>`).

Core source lives under `packages/core/src/` and follows the render pipeline, acquire → derive →
compose → render → persist:

- `domain`: shared types, enums, and tuned constants.
- `format`: pure formatting helpers (widths, number/duration/cost formatting).
- `sources` (acquire): payload, config, git, transcripts, fx, usage, clock, plus `sources/storage` (
  atomic writes and locks). Side-effecting reads live here.
- `derived` (derive): the classifier, the cost/pricing engine, and cross-session analytics.
- `compose` (compose): the statusline field set, the character line, and `compose/helpful` tips.
- `render` (render): layout, ANSI color, control-char stripping, width measurement.
- `packs`: the pack loader, validator, allowlist, registry, lint, and preview.
- `cli`: the render/classify/settings/uninstall/setup command logic (no UI).
- `tui`: the Ink setup UI and `tui/sections`.
- `web`: the browser render entry and Node-API shims that let the render pipeline run in-browser for
  the landing-page live demo (not shipped to npm).
- `bin`: the two executables.

```
Claude Code ──stdin JSON──▶  ccsidekick-render render  ──stdout ANSI──▶  status line (main agent only)
                                      ▲
three PostToolUse-family hooks ──ccsidekick-render classify──▶  events log + state  (disk only, no API, no tokens)
```

### Two binaries

- **`ccsidekick-render`** is the lean hot path. It carries the `render` and `classify` subcommands,
  pulls in no Ink or React, and runs under plain Node. Claude Code spawns it on every statusline
  tick and every tool call.
- **`ccsidekick`** is the user-facing entry. A bare invocation in a TTY launches the Ink setup UI (a
  first run — no `config.toml` yet — shows the guided wizard; a returning user gets the dashboard).
  `setup [flags]` configures and wires non-interactively (no Ink),`list <characters|themes|widgets>`
  prints valid values, and `uninstall` reverses the wiring. The TUI is the one place outside
  `tui/**` that may load Ink/React, and the bin imports it lazily so `setup`/`list`/`uninstall`
  never pull in the UI runtime.

State lives under `~/.claude/ccsidekick/` (honoring `CLAUDE_CONFIG_DIR`), partitioned per session
under `sessions/<id>/`. The cross-session analytics store (`analytics/`) and the per-file cost cache
are lock-guarded.

### Landing site

The `ccsidekick.krayong.com` landing page lives in `website/` (served files) with its build logic in
`scripts/website/*.ts`; it is deployed as a Cloudflare static-assets Worker, separate from the npm
packages. Most of `website/` is generated (gitignored) from committed sources: copy from
`scripts/website/site-content.ts`, tokens from `website/DESIGN.md`, and the live-demo render bundle
from `packages/core/src/web/**`. See `website/CLAUDE.md` before touching it. Run `bun run site:build`
to rebuild and `bun run site:serve` to preview.

## Load-bearing invariants

- **Pack art is a single 9×25 figure:** `art: readonly string[]` — one figure, its rows, keyed by
  nothing. There is no per-mood or multi-frame art; `pack.art` is read directly. Mood adds
  color-only effects (pulse/brighten/tint) that never shift a glyph, so the figure cannot strobe.
- **Figure box is fixed and lint-enforced:** the figure is at most 9 rows × 25 columns; an
  over-budget figure fails `pack:lint`. That authoring gate is separate from the render-time drop:
  below `MIN_RIGHT_WIDTH` the figure is dropped entirely and the statusline leads with a pack chip.
- **Packs are data, never executed code, and every pack is bundled.** The loader reads `pack.json`
  as JSON and never `require()`s pack code. Every pack ships with the engine as a runtime dependency
  (declared in `packages/core/package.json` and listed in `PACKS`), so a fresh install has every
  character; there is no install, browse, or download path. The default config runs random mode over
  the whole roster (empty `roster`); `spiderman` is only the fixed-mode default `name`. The
  loader still validates a pack name's segment (`allowlist.ts`) before resolving it, guarding
  `import.meta.resolve` against path traversal. A new pack is added by dropping it in
  `packages/packs/<name>` (passing `pack:lint`) and registering it in `PACKS`.
- **Art is sourced, never hand-drawn,** through the `ascii-art` image-to-ASCII skill, and every
  figure credits its artist in `attribution`.
- **Config is exactly** (tables in dashboard-section order): `schema_version`, then `[character]`
  (enabled, mode, name default `spiderman`, roster), `[theme]` (`name` default `character` — the
  Match

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krayong/ccsidekick](https://github.com/krayong/ccsidekick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
