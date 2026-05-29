---
trigger: always_on
description: FastHTML + MonsterUI portfolio site. Server-rendered Python, no JS framework. BigQuery-backed blog content. Auto-deploys to Google Cloud Run on push to `main`.
---

# gabriel-navarro-bio

FastHTML + MonsterUI portfolio site. Server-rendered Python, no JS framework. BigQuery-backed blog content. Auto-deploys to Google Cloud Run on push to `main`.

## Stack

- **Web framework**: FastHTML (`from fasthtml.common import *`) + MonsterUI (`from monsterui.all import *`), built on FrankenUI 2.
- **Server**: Uvicorn via `serve()` in `app.py`. Python 3.13 only.
- **Data**: BigQuery (`noble-office-299208.portfolio.gn-blog`) via `src/services/gcp/bigquery.py` REST client. No SQLAlchemy / ORM. Schema includes `body` (markdown) AND `body_html` (pre-rendered, post-lint HTML — render happens at submit time, not per request).
- **Validation**: Pydantic v2 (`src/services/blog_frontmatter.py` for `BlogFrontmatter` and `BlogRow` BQ payload model).
- **CSS**: hand-written in `src/styles/{_base,_layout,_components,_pages}.py`, concatenated into `FACTORY_CSS`. Tailwind preflight comes via FrankenUI; no Tailwind config of our own.
- **No React / TypeScript / Vite / Vitest** — server-rendered Python only.

## Project structure

- `app.py` — entry point: creates the FastHTML app, registers routes, calls `serve()`.
- `src/core/` — `app_factory.py` (theme + headers), `routes.py` (route registration; static-asset reorder for `/feed.xml`).
- `src/components/` — `base/` (Card, button helpers), `layout/` (StandardPage, navigation, footer).
- `src/features/` — page features grouped by route (hero, projects, cv, feed).
- `src/services/` — `gcp/bigquery.py`, `projects.py`, `blog_frontmatter.py`, `blog_lint.py` (auto-fix mistletoe foot-guns in inline-SVG markdown), `blog_render.py` (markdown → HTML at submit time + post-render validation).
- `src/cli/` — `python -m src.cli blog {validate|submit|update|disable|list}`.
- `src/styles/` — `_base.py`, `_layout.py`, `_components.py`, `_pages.py` concatenated into `FACTORY_CSS`. (`custom_css.py` is orphan dead code, not bundled.)
- `src/models/project.py` — `Project` dataclass with `from_dict` for BQ rows. Carries both `body` (markdown) and `body_html` (rendered); auto-computes `slug` via `python-slugify` if BQ row lacks one.
- `tests/` — pytest with `mock_bq` fixture in `conftest.py`.
- `assets/blogs/*.md` — markdown blog sources (legacy `@{...}` or YAML frontmatter; both supported by parser).

## Setup / dev loop

- **Install**: `pip install -e ".[dev]"` (`requirements.txt` was deleted in Epic D — `pyproject.toml` is the source of truth).
- **Run**: `make dev` or `python app.py --port 8080`.
- **Test**: `make test` or `pytest -q`.
- **Lint**: `make lint` (`ruff check . && ruff format --check .`).
- **After pulling main**, if any new runtime deps were added, re-run `pip install -e .` — symptom otherwise is `ModuleNotFoundError` on app start.

## Coding conventions

- Python 3.13, type hints, `ruff` format/lint, Google-style docstrings.
- Format on save. Lint before commit. Pre-commit runs ruff + pytest automatically.
- Star imports are idiomatic for FastHTML/MonsterUI: `from fasthtml.common import *`, `from monsterui.all import *`. Ruff is configured to ignore `F403`/`F405`.

## Layout & CSS conventions

- **No inline `style=`** in `src/features/**`. Acceptance check: `grep -r "style=" src/features/` must return empty.
- **Use the `Card` primitive** (`src/components/base/card.py`) for any bordered container, not ad-hoc `Div(style="border: ...")`.
- **Use button helpers** (`button_primary|outline|ghost`), not inline `<A>` CTAs.
- **MonsterUI documented patterns only**: `NavBar(*A_items, brand=...)` with positional `A(...)` args. Do NOT use `Ul(Li(A(...)), cls="uk-navbar-nav")` — FrankenUI 2 dropped many UIkit classes (`.uk-navbar-nav` is a no-op).
- **`100vw` is a trap** — includes scrollbar width; use `100%` unless you specifically need scrollbar-included.
- **`overflow-x: clip` not `hidden`** when locking horizontal scroll — `hidden` creates a new containing block and breaks `position: sticky`.
- **Category colors** live as `--cat-{omics|ml|infra|viz}` CSS vars in `_base.py`. Never hardcode the hex values elsewhere; use `category_class(tag)` from `src.config.settings` to map a tag to its `cat-*` class name.

## Testing patterns

- `tests/conftest.py::mock_bq` patches `src.services.projects.BigQueryClient` (at the **import site**, not the definition site `src.services.gcp.bigquery.BigQueryClient`).
- Use `to_xml(el)` as a top-level function from `fasthtml.common`, NOT `el.to_xml()` — FastHTML's `__getattr__` resolves the latter to `None`.
- TDD red-bar: mark intentionally-failing tests `@pytest.mark.xfail(reason="ticket-id implements...")`; remove the marker when the implementing ticket lands.
- Don't double-mock `Project.from_dict` — let the model exercise itself; only mock at the BQ boundary.

## Git workflow

- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`, `build:`, `ci:`, `style:`.
- Feature branches off `main`. PRs require passing CI.
- Every commit ends with footer:
  ```
  Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
  ```
- NEVER `--amend` — make a NEW commit per fix (Git Safety Protocol).
- NEVER push to `main` directly. NEVER use `--no-verify`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabenavarro/gabriel-navarro-bio](https://github.com/gabenavarro/gabriel-navarro-bio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
