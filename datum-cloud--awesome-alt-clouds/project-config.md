---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`awesome-alt-clouds` is a curated "awesome list" of 400+ alternative cloud providers, published as an Astro static site at [alt-cloud.org](https://www.alt-cloud.org). Its distinguishing feature: submissions are evaluated and merged by an automated GitHub Actions + Python + Claude API pipeline, not just by hand.

**`README.md` is the single source of truth.** Every other artifact — `public/clouds.json`, `public/llms.txt`/`llms-full.txt`, and the website itself — is regenerated from it on every merge to `main`. Never hand-edit `public/clouds.json` or `public/llms*.txt`; they get overwritten by `scripts/parse_readme_to_json.py` / `scripts/generate_llms.py` in `deploy-pages.yml`.

For the full pipeline breakdown (submission → evaluation → PR → deploy), read `analysis/PROJECT_ANALYSIS.md` first — it's a complete architecture doc and stays more current than trying to re-derive the flow from scripts alone.

## Commands

```bash
npm install
npm run dev            # Astro dev server, http://localhost:4321
npm run build          # astro build -> dist/ (postbuild rewrites relative paths for GitHub Pages)
npm run preview
npm run check           # astro check
npm run lint            # eslint . && astro check
npm run lint:fix
npm run format           # prettier --write .
npm run format:check
```

Python (submission pipeline scripts + tests — not run in any CI workflow, run manually):

```bash
python3 -m pip install --user --break-system-packages -r requirements-dev.txt
pytest tests/
ruff check scripts/ tests/
ruff format scripts/ tests/
```

Single test: `pytest tests/test_evaluate_submission.py::TestClassName::test_name`

Regenerate derived files locally (mirrors `deploy-pages.yml`):

```bash
python scripts/parse_readme_to_json.py README.md public/clouds.json
python scripts/generate_llms.py public/clouds.json public/
python scripts/generate_watchlist_json.py
```

## Architecture

### Astro frontend (`src/`)

- `src/pages/index.astro` — homepage card grid; `[slug].astro` — per-cloud detail page; `categories/[slug].astro` — 23 static category pages; `compare.astro` — side-by-side comparison (localStorage basket, max 3); `blog/`, `submit/`, `watchlist/`.
- `src/content/clouds/<slug>.mdx` — per-provider detail pages, schema in `src/content.config.ts`. Key frontmatter field: `status: "draft" | "reviewed"` (default `"draft"`).
- `src/content/blog/<slug>.md` — editorial posts, frontmatter includes `draft: boolean` (default `false`).
- `src/lib/profile.ts` / `src/lib/blog.ts` — the **publish gate**: `getPublishableProfiles()` / `getPublishablePosts()` filter drafts out unless the build is in preview mode. This same filter drives both route generation (`getStaticPaths()`) and homepage/search link visibility, so a route and its links never drift apart. Don't build a new draft-aware feature without going through this gate.
- `src/lib/clouds.ts` — canonical `slugify()`. Mirrored byte-for-byte in `scripts/lib/slugify.py`; MDX filenames in `src/content/clouds/` must match the slug this produces. Keep both in sync if you touch either.
- `src/lib/categories.ts` — the 23-category taxonomy + slugs. Must stay in sync with `scripts/evaluate_submission.py:CATEGORIES` and `scripts/generate_llms.py:_CAT_DESCRIPTIONS` (three places, one taxonomy).

### Preview vs. production build

`site.config.mjs` is the single deploy-profile switch (`preview: true | false`, `blockSearchBots`). It controls, via `astro.config.mjs`'s `__SITE_PREVIEW__` define and `src/lib/site.ts`'s `sitePreview` export:

- base path (`/awesome-alt-clouds/` on preview vs. site root in production)
- whether `status: draft` MDX / `draft: true` posts get built at all
- robots.txt / meta-robots crawler blocking

Always read `sitePreview` from `src/lib/site.ts`, never re-import `site.config.mjs` directly at runtime — Vite inlines the value at config-load time and a live re-import risks a stale value.

### Submission pipeline (`scripts/` + `.github/workflows/`)

```
submit form → GitHub issue (label: submission)
  → check_duplicates.py (fail-open guard against clouds.json + open issues)
  → evaluate_submission.py (3-stage fetch cascade: Jina Reader → requests → Claude web_search)
     → 3 criteria checks (pricing / self-service / SLA-status) + Claude-generated name/description/category
  → create_submission_pr.py (alphabetical insert into README.md; also drafts an MDX profile via generate_cloud_profile.py in the same commit)
  → admin merge (or /approve override) → deploy-pages.yml regenerates clouds.json + llms*.txt + astro build
```

- Multi-URL issues are split by `split_submission.py` into per-URL child issues before the same evaluate → PR flow runs on each.
- Inclusion needs ≥2 of 3 criteria (transparent pricing, self-service signup, public SLA/status page): 🟢 = 3/3, 🟡 = 2/3.
- Fetch cascade lives in `scripts/lib/fetcher.py`, shared by `evaluate_submission.py` and `generate_cloud_profile.py` — never call `requests.get()` directly when evaluating a candidate site, use `fetch_page_with_fallback()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datum-cloud/awesome-alt-clouds](https://github.com/datum-cloud/awesome-alt-clouds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
