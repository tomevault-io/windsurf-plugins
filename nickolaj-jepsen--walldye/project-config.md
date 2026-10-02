---
trigger: always_on
description: walldye is a catalog of procedural SVG wallpapers. Each piece is a small Python script, one `@design` function that draws with the `walldye` library in symbolic theme colors, optionally with a few named variants. `walldye build` draws it once per version, screen shape and regime, writes SVG templates, which CI builds and never commits, and records every color in them as a linear mix of three seed colors (bg, fg, accent), read off the formula it was drawn with. The Astro site at walldye.com uses 
---

# walldye

walldye is a catalog of procedural SVG wallpapers. Each piece is a small Python script, one `@design` function that draws with the `walldye` library in symbolic theme colors, optionally with a few named variants. `walldye build` draws it once per version, screen shape and regime, writes SVG templates, which CI builds and never commits, and records every color in them as a linear mix of three seed colors (bg, fg, accent), read off the formula it was drawn with. The Astro site at walldye.com uses those mixes to show every piece in the visitor's colors and to export it as SVG, PNG, WebP or JPEG.

## Docs

- `docs/architecture.md`: how walldye works and why. Update it in the same change when a decision changes.
- `docs/wallpapers.md`: a wallpaper's folder and meta.yaml, the copy rules, licensing and what a piece may draw.
- `docs/api.md`: the design API, build files and CLI. The docstrings in `walldye/` hold the exact semantics; when the two disagree, fix one of them in the same change.
- `docs/site.md`: the site's visual system, components and voice.
- `docs/deploy.md`: CI, Cloudflare and the going-live checklist.
- `src/client/DOM.md`: the hooks the server-rendered pages give the client modules.

## Writing docs and comments

- Docs describe the current system in as few words as it takes: no plans, history, open tasks or lists of what the tests cover, and nothing the code or a docstring already says. Each fact lives in one doc; the others link to it.
- Comments say only what the code can't: intent, a constraint, a gotcha, a tradeoff, units or bounds. One line where possible; no narrating the line below, no change history ("v1", "used to", "replaces").
- Docstrings state the contract (behavior, non-obvious parameters, return value, errors, invariants), not the signature or the callers.
- Comments and docstrings stand on their own and never cite the docs or their sections.
- No AI tropes: em dashes, "not X, it's Y", "serves as", "it's worth noting", magic adverbs (quietly, deeply), bold-first bullets, signposted conclusions.

## Layout

- `walldye/`: the library designs import (`walldye`, `walldye.geom`, `walldye.field`, `walldye.pixel`; the `_*.py` modules implement them), plus the CLI in `walldye/tools/`. The code outside `tools/` feeds the render-lib hash, so changing it (not its comments or docstrings) makes the next build re-check every piece's probes.
- `wallpapers/<slug>/`: `design.py`, `meta.yaml` and the optional `data/` are written by hand; `build/` is generated and gitignored, with named variants in `build/<variant>/`. Legacy pieces have `source.svg` and `palette.yaml` instead of a script. `wallpapers/index.json` is generated and gitignored too, and `wallpapers/pyrefly.toml` sets the type-check level for designs.
- `taxonomy.yaml`: the allowed facet values for meta.yaml, each with the words the site shows for it, and the credit name of each model id. Only `walldye review` adds facet values.
- `featured.yaml`: the pieces the index opens on, in order, chosen by the owner.
- `src/`: the site. `src/lib/` is pure code any side may import (no DOM, Node or Astro runtime): the TypeScript ports of the theme, recolor and tokenizer code, with the fixtures shared with pytest in `src/lib/__fixtures__/`, and `src/lib/labels.ts`, the words visitors see for the facets and licenses (taxonomy.yaml has those for facet values and models). `src/client/` runs only in the browser (one `page.ts` per page, plus the theme boot), `src/server/` only at build time (the collection in `src/server/catalog/`, meta.yaml's schema helpers and taxonomy.yaml).
- `tests/`: `python/` (pytest: `core/`, `helpers/`, `tools/`, and `fixtures/` with the synthetic designs), `unit/` (vitest, self-contained), `parity/` (vitest against the Python output), `e2e/` (Playwright), and `fixtures/`, the gitignored Python renders and fixtures the parity tests read.
- `scripts/fixtures/`: `regen.py` writes `tests/fixtures/` from a build.
- `scripts/fonts/`: rebuilds the subset fonts in `src/assets/fonts/`.
- `scripts/promo/`: records the site's promo loop into `promo/`, gitignored: the index's themes and shapes, the plate carried to its page, and a Download.
- `scripts/views/`: fetches the daily page views that `.github/workflows/views.yml` stores on the `stats` branch; CI checks that branch out as `stats/`, gitignored, for the index's view sorts.
- `flake.nix`: the Nix package, `mkWallpaper` and the dev shell. The Python environment comes from `uv.lock` through uv2nix.
- `infra/www-redirect/`: the Worker that sends www.walldye.com to the apex, deployed by hand.
- `.claude/skills/walldye/`: the skill for designing the wallpapers the owner names, or reworking one. `.claude/workflows/wallpaper-batch.js` invents a batch of new ones from research; `.claude/workflows/README.md` explains its arguments.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickolaj-jepsen/walldye](https://github.com/nickolaj-jepsen/walldye) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
