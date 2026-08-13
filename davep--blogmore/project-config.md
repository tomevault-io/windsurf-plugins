---
trigger: always_on
description: All source lives in `src/blogmore/`. Key modules and their responsibilities:
---

# Agent Instructions for BlogMore

## Codebase architecture

All source lives in `src/blogmore/`. Key modules and their responsibilities:

| Module | Responsibility |
|---|---|
| `__main__.py` / `cli.py` | Entry point; CLI argument parsing |
| `config.py` | Loads and merges `blogmore.yaml` into a runtime config object; `parse_site_config_from_dict` is the single source of truth for YAML→`SiteConfig` field mapping |
| `site_config.py` | `SiteConfig` dataclass — the validated, typed site configuration |
| `generator/` | Core static site generator (see sub-package table below) |
| `parser.py` | Markdown + frontmatter parser; produces `Post` and `Page` objects |
| `renderer.py` | Jinja2 template rendering |
| `publisher.py` | Git-based publishing (`blogmore publish`) |
| `server.py` | Local dev server with file watching (`blogmore serve`) |
| `sitemap.py` | XML sitemap generation |
| `feeds.py` | RSS and Atom feed generation |
| `search.py` | Client-side search index generation |
| `icons.py` | Favicons and touch icons from a single source image |
| `fontawesome.py` | FontAwesome CSS tree-shaking/optimisation |
| `post_path.py` | Configurable output path resolution for posts |
| `pagination_path.py` | Pagination path resolution for configurable index page output paths |
| `content_path.py` | Shared path-resolution utilities for content output paths (used by page_path and post_path) |
| `clean_url.py` | Clean URL transformation utilities (removes index.html from URLs when enabled) |
| `page_path.py` | Page path resolution for configurable output file paths |
| `backlinks.py` | Internal link analysis and backlink map generation |
| `calendar.py` | Calendar grid calculation and data structures |
| `graph.py` | Post relationship graph data generation (JSON) |
| `stats.py` | Blog statistics computation (word counts, top tags, etc.) |
| `comment_invite.py` | mailto: URL generation for "Invite comments" links |
| `code_styles.py` | Pygments-based CSS generation for code blocks |
| `stop_words.py` | Standard English stop words and utilities |
| `utils.py` | Shared utility helpers |

The `generator/` sub-package (`src/blogmore/generator/`) breaks the site
generator into focused modules and component classes:

| Module | Responsibility |
|---|---|
| `generator/constants.py` | Filename constants, `TAG_DIR`, `CATEGORY_DIR`, `PAGE_SPECIFIC_CSS` |
| `generator/utils.py` | `minified_filename`, `paginate_posts` |
| `generator/grouping.py` | Post grouping by tag/category; word-cloud font-size interpolation |
| `generator/paths.py` | Pagination paths, canonical URLs, output path resolution, sidebar filtering |
| `generator/html.py` | HTML writing and minification |
| `generator/assets.py` | `AssetManager` — icons, FontAwesome CSS, static file copying, extras, CSS/JS minification |
| `generator/context.py` | `ContextBuilder` — global template context, asset URL helpers, cache-busting |
| `generator/pages.py` | `PageGenerator` — core page generation (post, static page, index, archive) |
| `generator/listings.py` | `ListingGenerator` — date archives and tag/category paginated listings |
| `generator/features.py` | `FeatureGenerator` — feeds, search, stats, calendar, graph, sitemap |
| `generator/site.py` | `SiteGenerator` — top-level orchestration |
| `generator/__init__.py` | Backward-compatible re-exports |

`SiteGenerator` composes these components:

```
SiteGenerator
  ├── AssetManager
  ├── ContextBuilder
  ├── PageGenerator
  ├── ListingGenerator
  └── FeatureGenerator
```

The `markdown/` sub-package (`src/blogmore/markdown/`) groups all custom
Markdown extensions and utility modules:

| Module | Responsibility |
|---|---|
| `markdown/admonitions.py` | Markdown extension: GitHub-style `> [!TYPE]` admonitions |
| `markdown/external_links.py` | Markdown extension: opens external links in a new tab |
| `markdown/heading_anchors.py` | Markdown extension: hover anchor links on headings |
| `markdown/strikethrough.py` | Markdown extension: `~~strikethrough~~` syntax |
| `markdown/plain_text.py` | Markdown-to-plain-text conversion |
| `markdown/first_paragraph.py` | Logic for extracting the first meaningful paragraph as plain text |
| `markdown/__init__.py` | **Single source of truth for the custom extension set** via `create_custom_extensions` |

Any new Markdown extensions must be added as a new module inside
`src/blogmore/markdown/`, registered in `create_custom_extensions` in
`src/blogmore/markdown/__init__.py`, and imported in `parser.py`.

Templates live in `src/blogmore/templates/`; the stylesheet is
`src/blogmore/templates/static/style.css`.

When adding a self-contained unit of new functionality, create a new
appropriately-named module rather than growing an existing large file.

## Code style

- Always write full type hints that pass `mypy` in strict mode.
- If a third-party library lacks type hints, search for a companion type-stub
  package (e.g. `types-*`) before adding `# type: ignore` comments.
- Always generate full Google-style docstrings for every module, class,
  method, and function. Do *not* include type information in docstrings.
- All inline code and cross-references in docstrings **must** use mkdocstrings-compatible Markdown style:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davep/blogmore](https://github.com/davep/blogmore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
