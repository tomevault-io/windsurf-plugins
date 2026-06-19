---
trigger: always_on
description: Guidance for Claude Code (and other AI assistants) working in this repo.
---

# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repo.

## What this project is

`marimo-book` is a Jupyter-Book-style static site generator for marimo
notebooks. It reads a directory of `.md` and marimo `.py` files plus a
`book.yml` config and emits a polished documentation site built on
Material for MkDocs (and, eventually, on its Rust successor zensical —
the architecture is shell-agnostic).

## Architecture in one paragraph

`book.yml` → preprocessor (`src/marimo_book/preprocessor.py`) → staged
tree at `_site_src/` (Markdown + inline HTML, plus a generated
`mkdocs.yml`) → `mkdocs build` → `_site/`. The preprocessor never
shells out to mkdocs; it just emits artifacts mkdocs can consume. This
keeps the shell swappable. The CLI (`src/marimo_book/cli.py`) glues
the preprocessor + `mkdocs build`/`serve`.

## Common commands

```bash
# Setup (in repo)
uv pip install -e '.[dev,linkcheck,social,autorefs]'

# Tests + lint
pytest -q
ruff check src/ tests/
ruff format --check src/ tests/

# Build the self-hosted docs (this repo's own book)
marimo-book build -b docs/book.yml

# Live-reload dev server for the docs
marimo-book serve -b docs/book.yml   # http://127.0.0.1:8000/marimo-book/

# Build a brand-new book scaffold
marimo-book new ~/my-book
cd ~/my-book && marimo-book serve
```

When `marimo-book serve` won't pick up CSS changes after a `build`,
it's the in-memory mkdocs cache. **Kill and restart serve** rather
than waiting for auto-reload.

## Release flow (the important one)

Releases are tag-driven via `hatch-vcs`. There is **no `version` field
in `pyproject.toml`** — the version is the latest `v*` git tag.

`.github/workflows/publish.yml` triggers on **`push: tags: ["v*"]`**
(not on publishing a GitHub Release). So the act that ships a release
is **pushing a `v*` tag**; `hatch-vcs` reads the tag, builds a wheel
versioned exactly to it (e.g. `v0.1.18` → `0.1.18`), and ships it to
PyPI via OIDC Trusted Publisher.

To cut a release:

1. Open a tiny PR that dates the `[Unreleased]` section in
   `CHANGELOG.md` to today (one-line change, e.g.
   `## [0.1.18] — YYYY-MM-DD`). Merge it.
2. Tag merged `main` at that commit **via `gh release create`** — this
   creates+pushes the `v*` tag (which fires `publish.yml` → PyPI) **and**
   creates the matching GitHub Release object, so the repo's Releases page
   stays current:

   ```bash
   git checkout main && git pull
   # Notes = that version's CHANGELOG section.
   gh release create v0.1.21 --target main --title v0.1.21 \
     --notes "$(awk '/## \[0\.1\.21\]/{f=1;next} /^## \[/{if(f)exit} f' CHANGELOG.md)"
   ```

   Watch the publish run: `gh run watch --repo ljchang/marimo-book`
   (or the Actions tab). A bare `git tag … && git push origin v0.1.21`
   still works and still publishes to PyPI — but it leaves **no** GitHub
   Release object, which is why the Releases page used to lag. Prefer
   `gh release create`.

That's it. **Never push version edits directly to main**, and never
push a `v*` tag you don't intend to publish — the tag *is* the
release trigger. The version lives in exactly one place: the git tag.

**Note (2026-06): `gh release create` is now the standard.** Earlier
releases (`v0.1.12`–`v0.1.20`) were cut by bare tag push and had no
GitHub Release object; they've since been backfilled from the CHANGELOG,
so every `v0.1.x` tag now has a Release. Use `gh release create` going
forward so this never drifts again. The old `release-drafter` draft
(stuck at `v0.1.11`) is **not** part of the live flow — ignore it.

See `PUBLISHING.md` for full detail (one-time PyPI setup, label
conventions for release-drafter categorisation, yanking, etc.).

## Branch protection / direct main pushes

Pushes to `main` are blocked by the harness for safety. All work goes
through PRs. CI must be green before merge:

- `test (3.11/3.12/3.13)` — `pytest` + `ruff check` + `ruff format --check`
- `build` — sdist + wheel build with required-files check
- `docs` — `marimo-book build -b docs/book.yml --strict`

If a PR introduces a new optional extra (like `[autorefs]`), update
both `.github/workflows/ci.yml` and `.github/workflows/docs.yml` to
install it (the docs job needs every extra the docs site uses).

## Feature flags users can opt into via `book.yml`

| Flag | Effect | Extra needed |
|---|---|---|
| `social_cards: true` | Material's `social` plugin auto-generates per-page OG/Twitter card PNGs | `marimo-book[social]` (pulls Pillow + cairosvg, ~20 MB; needs system `libcairo2 libpango-1.0-0 libpangocairo-1.0-0`) |
| `cross_references: true` | `mkdocs-autorefs` resolves `[Heading text][]` to whichever page has that heading (MyST `{ref}` analog) | `marimo-book[autorefs]` |
| `check_external_links: true` | `htmlproofer` validates external URLs at build (slow; CI-only) | `marimo-book[linkcheck]` |
| `include_changelog: true` | Preprocessor copies `CHANGELOG.md` from book root (or its parent) into the staged tree and appends a "Changelog" entry to the nav | None |
| `pdf_export: true` | `mkdocs-with-pdf` renders the whole book to `_site/pdf/book.pdf` via WeasyPrint and adds a "Download PDF" link to the footer | `marimo-book[pdf]` (same cairo/pango system deps as `[social]`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ljchang/marimo-book](https://github.com/ljchang/marimo-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
