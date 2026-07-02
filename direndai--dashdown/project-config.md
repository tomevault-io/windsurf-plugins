---
trigger: always_on
description: validates a token **scoped to that exact resource**.
---

# CLAUDE.md

This file orients coding agents (and humans) working on the **Dashdown framework** itself. For
*authoring a dashboard* with Dashdown, read the user-facing docs in `docs/` (run them with
`dashdown serve docs`) — this file is about the internals of the package.

## What this is

Dashdown renders Markdown files (with embedded SQL and component tags) as interactive analytics
dashboards. No build step, no JS framework, no npm — the frontend is hand-written ES modules served
as static files; the backend is a FastAPI app. Distributed as the `dashdown` pip package with a CLI
entry point.

## Commands

```bash
uv run pytest tests/ -v              # run the full suite
uv run pytest tests/test_attrs.py    # single file
uv run pytest tests/test_pipeline.py::TestSubstituteParams -v   # single class
uv run pytest -k "injection"         # match by test name

pip install -e .                     # editable install (or: uv sync)
dashdown serve docs                  # run the docs project as a live dashboard at http://127.0.0.1:8000
dashdown serve . --port 8001 --no-watch
dashdown query "SELECT * FROM sales LIMIT 5" -p docs -c main   # probe a connector / inspect data (table|json|csv)
dashdown components                  # introspected catalog: component attrs + connector config keys (table|json; --connectors)
dashdown new my-dashboard            # scaffold a new project
dashdown build docs --out dist       # static export (pre-rendered HTML + data JSON)
```

There is no configured linter/formatter and no frontend build/test tooling — JS is shipped as-is.
The `docs/` project doubles as an end-to-end integration fixture; serve it to verify rendering
changes by hand.

## Two distinct domains

Keep clear about which side of the wire you're on:

1. **The framework** (`dashdown/`) — the Python package + static JS that *we* ship.
2. **A user's project** — a directory with `dashdown.yaml`, `sources.yaml`, `pages/*.md`, `data/`,
   `components/`, `assets/`. The CLI points the framework at one of these. (The project's served
   asset folder is `assets/` — images, downloads, `custom.css`, branding logo/favicon — mounted at
   `/assets/`; pages may also reference files co-located next to their `.md`. See
   `render/pipeline.py::_rewrite_asset_urls`.)

## Render pipeline (the core flow)

A page request flows through `server.py` → `render/pipeline.py` → `render/markdown.py` +
`render/components.py`. Read these four files together before changing rendering behavior.

1. `parse_markdown()` (`render/markdown.py`) splits the `.md` into: YAML frontmatter, HTML body, and
   a list of `QuerySpec` (from `:::query name=… connector=…` container directives). The SQL inside a
   query block is **collected, not executed**, and stripped from the HTML output. `build_md()` is
   CommonMark + tables + GitHub-flavored extensions (strikethrough, task lists, footnotes, deflists,
   `h2/h3` heading anchors) + `:::note`/`:::tip`/`:::info`/`:::warning`/`:::danger` callout containers
   (same `:::` machinery as `:::query`, distinct first words so they don't collide). Fenced code is
   highlighted **server-side** via Pygments through markdown-it's `highlight` option (`highlight_code`
   emits a `<pre class="dashdown-code" data-lang=…>` shell — unknown langs fall back to a plain escaped
   block in the same shell; `mermaid` is special-cased to an explicit `dashdown-mermaid` marker block,
   never highlighted, which the client upgrades to a diagram). Highlighting is static HTML, so it ships
   in `dashdown build` exports/embeds with no client JS. Styling is plain CSS under `.dashdown-prose`
   in `static/dashdown.css`. A copy-to-clipboard button is layered on **client-side**
   (`static/components/copy_code.js` wraps each non-mermaid `<pre class="dashdown-code">` and injects
   the button) — a pure progressive enhancement, so this server-side shell stays untouched. The
   separate `render_markdown_text()` (untrusted `<Ask />` LLM answers)
   stays minimal and `html: False` — these page extensions are deliberately **not** applied there.
2. `render_components()` (`render/components.py`) scans the HTML for PascalCase tags
   (`<LineChart .../>`, `<Table>…</Table>`) with a stack-based parser and replaces each with the
   registered component's `render()` output. Unknown tags / render errors become inline `_error_card`
   divs rather than 500s.
3. `render_page()` (`render/pipeline.py`) registers each query def into a **module-global cache**
   keyed by `(name, connector)` and emits HTML with empty datasets. The page's *effective* query set
   is its inline `:::query` specs **plus** any shared-library queries its components reference by name
   (precedence local → library → unresolved).

**Key architectural decision: queries never run server-side during page render.** The page ships
instantly with no data; the browser then calls `GET /_dashdown/api/data/{query_name}?…&_connector=…`
per query. This is why query defs live in a process-global cache (`_query_def_cache` in
`render/pipeline.py`) — the data API is a *separate* request from the page render and needs to look
the SQL back up. Don't "fix" this into a request-local registry: a prior race-condition bug came from
exactly that.

## SQL parameter substitution & injection


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DirendAI/dashdown](https://github.com/DirendAI/dashdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
