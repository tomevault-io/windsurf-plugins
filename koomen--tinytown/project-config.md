---
trigger: always_on
description: Notes for coding agents working in this repository. Read
---

# CLAUDE.md

Notes for coding agents working in this repository. Read
`docs/ARCHITECTURE.md` first: it is the contract every module follows. The CLI
(`./town <verb> --help`) is the truth for flags; the docs describe intent.

## Setup

```sh
python3 -m venv .venv && .venv/bin/pip install -e .   # Python >= 3.10; pillow, websocket-client
./town browser setup                                  # private headless Chromium -> runs/headless-browser/ (render, bake, browser tests)
node --version                                        # >= 22 for bake and tests
codex login                                           # only for `town author` (OpenAI Codex CLI)
```

`./town` runs `python -B -m tinytown` with `.venv/bin/python` when present
(`PIPELINE_PYTHON` overrides). `serve`, `build`, `stage`, `bake --check`,
`status`, `plan`, `lint` need only the standard library.

## Verbs

| Verb | One line |
| --- | --- |
| `fetch <site> [--center LAT,LON --size W,H --title T] [--no-satellite] [--force] [--aerials [ID…]]` | OSM, USGS elevation, Esri imagery into `data/<site>/source/`; crops aerials |
| `scope <site> [--ids…] [--ids-file F] [--bounds S,W,N,E] [--exclude…] [--source SITE]` | freeze `sites/<site>/scope.json`; `--source` seeds a site from another's downloads |
| `build <site>` | `source/` + `overrides.json` -> `data/<site>/site.json` (milliseconds) |
| `refs <site> [ids…] [--all] [--list F] [--faces=+u,-v\|all\|road] [--force] [--missing] [--aerials] [--web] [--extra-views N]` | Street View fronts, aerials, model packet into `buildings/<id>/` |
| `brief <site> [ids…] [--list F]` | `buildings/<id>/brief.md` + `footprint.png` |
| `plan <site> [--limit N] [--out F] [--json]` | what still needs research/authoring/review, by priority |
| `render <site> <id> [--face=F]… [--dist M] [--iso] [--with ID…] [--no-bp] [--compare] [--force] [--trees]` | screenshot a draft through the viewer; `compare-<face>.png` beside the photo |
| `lint <site> [ids…] [--merged] [-q]` | blueprint lint (drafts, or `overrides.json` with `--merged`); errors exit 1 |
| `review <site> ids… [--stage massing\|detail] [--record]` | non-model review: lint + geometry audit + render evidence -> `review.json` |
| `author <site> [ids…] [--all] [--reauthor ID…] [--accept] [--force] [--dry-run] [--workers N] [--max-tokens N] [--max-seconds S]` | stages 3–6 per building: references, author, render, review, <= 2 repairs, scene critique, accept |
| `accept <site> [ids…] [--all-reviewed] [--force] [--no-rebuild]` | reviewed drafts -> `overrides.json`, then `build`; the only writer of blueprints |
| `status <site> [--ids…]` | derived per-building status |
| `bake <site> [--check] [--surfaces-only\|--stream-only]`, `bake --viewer [--check]` | surfaces + stream chunks for a site; `?v=` stamps in `index.html` |
| `deploy [--target avon\|chautauqua\|all] [--no-check]` | stage `dist/<target>/` after `bake --check` and viewer checks |
| `serve [--port 8734] [--dist [TARGET]]` | dev server: `/`, `/avon`, `/chautauqua`, `/?site=<name>`; or a built dist |
| `verify <target> <domain> [site]` | live files match `dist/<target>/` |
| `browser setup\|status\|cleanup\|stop` | the private headless Chromium |
| `migrate <site>… [--dry-run]` | pre-2026-09 layout -> current layout |

## Edit -> rebuild loops

- **Viewer or generator (`src/`)**: `./town bake <site>` for every affected site
  (surfaces are fingerprinted on all `src/*.js`; streams on the generator
  modules), then `./town bake --viewer`, then commit the regenerated `data/`
  and `index.html`. `./town bake <site> --check` says what is stale.
- **Authored data (`data/<site>/overrides.json`, `sites/<site>/landmarks.json`,
  `scope.json`)**: `./town build <site>` then `./town bake <site>`.
- **A blueprint**: edit `buildings/<id>/draft.json` -> `town lint` ->
  `town render --compare` -> `town review --record` -> `town accept` (rebuilds)
  -> `town bake`. Or `town author <site> --reauthor ID --accept`.
- **Docs the model reads**: `docs/MINIATURE_KIT.md` is sent verbatim in every
  author prompt; `docs/miniature_examples.json` (falling back to
  `pipeline/miniature_examples.json`) supplies the style examples.

## Where things live

| Path | What |
| --- | --- |
| `tinytown/<stage>.py` | one module per stage; each with verbs exposes `register(subparsers)`; `cli.py` maps verbs to modules |
| `tinytown/paths.py`, `config.py`, `state.py` | every filesystem path; site/deploy config and derived routes; fingerprints, atomic JSON, status derivation |
| `tinytown/plugins/<site>.py` | per-site hooks: `extra_sources`, `landmarks`, `outline`, `refine_building`, `scope_filter` |
| `tinytown/web/` | `precompute.html` (surfaces), `prepare_streaming.mjs` + `stream-export.*` (chunks) |
| `index.html`, `src/` | the viewer, served as-is |
| `sites/<site>/site.json` | title, description, domain, `deploy` placements, `plugin`, `scope`, `landmarks`, `outline`, `social_image` |
| `sites/deploy.json` | target -> `{dist, wrangler}` |
| `data/<site>/source/` | stage-1 inputs (`satellite.jpg` gitignored; cache in `data/.town-cache/`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koomen/tinytown](https://github.com/koomen/tinytown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
