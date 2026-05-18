---
trigger: always_on
description: - Minimal fix: prefix `site.baseurl` to search result links in `docs/_layouts/default.html` and Enter key navigation.
---

# Agents Notes — repo_posts (Oct 26, 2025)

- Minimal fix: prefix `site.baseurl` to search result links in `docs/_layouts/default.html` and Enter key navigation.
- Tests added: `tests/test_search_links_baseurl.py` to lock the behavior (anchor href + Enter navigation).
- CI/Deploy: Actions running; RSS smoke check passed; Pages deploy currently in progress and typically turns green within a minute after build.

Keep in mind
- Search JSON (`tools/generate_search_index.py`) keeps `u` as path like `/YYYY/MM/DD/slug.html`; client JS must always prefix `{{ site.baseurl }}`.
- Contribution policy lives in `README.md` only (not shown on the site header).

Oct 26, 2025 — Autoposter/Git Push
- Symptom: “new posts not appearing”. Root cause was non–fast‑forward push failures in the local publish step; pushes failed so Pages didn’t deploy.
- Fix: GitHubPublisher now fetches + pull --rebase and pushes with `--force-with-lease` before publishing.
- Behavior: autoposter continues even if GitHub push fails; it now logs an error and proceeds.

Next small items (safe to batch later)
- Optional: add `/` shortcut to focus the search input.
- Optional: render result title instead of slug (would require adding `title` to the index).

Added tests — Oct 26, 2025
- `tests/test_layout_basic.py`: blocks two-column grid; ensures per-post image and index link.
- `tests/test_layout_more.py`: scroll-margin rule, related block present, home image links to post, key handlers in search JS.
- `tests/test_search_links_baseurl.py`: search links prefixed with baseurl and Enter uses same.

CI — Oct 26, 2025
- Extended `rss-smoke.yml` to also curl a sample post, assert image + "View on index" are present, and verify `assets/search-index.json` and `assets/css/site.css` are reachable after a successful Pages deploy.

UX — Oct 26, 2025
- Added subtle page fade-in (0.15s) with reduced-motion guard.
- Homepage shows only the original repo slug/title inside the post content; removed the extra homepage title link above.
- Tests updated: `tests/test_title_link_to_post.py` now asserts absence of the extra title and of the hide rule.

Maintainability — Oct 26, 2025
- Extracted inline search script into `docs/assets/js/search.js` (with Liquid front matter) and moved the inline RSS badge styles into `site.css`.
- Updated tests to read from `search.js` and assert layout references it.

Next Steps — Oct 26, 2025
1) Add "/" shortcut to focus search input (tiny JS + 1 test). [Recommended]
2) Render titles in search results (extend `generate_search_index.py` with `title`, adjust `search.js`; add 1–2 tests).
3) Add workflow `concurrency` to Deploy to avoid occasional "in progress" Pages conflicts.
4) Limit homepage to recent N posts and add a simple Archive page to cut initial load (keep all posts online).
5) Optional: add max content width (e.g., 900px) for readability; add 1 CSS test.

Proposed Continuation — Oct 26, 2025 (later)
1) Search results show titles
   - Change: emit `title` in `tools/generate_search_index.py`; render titles in `assets/js/search.js`.
   - Tests: assert `title` appears and is highlighted.
2) Max content width for readability
   - Change: `section{max-width:900px;margin:0 auto}` in `site.css`.
   - Tests: check rule presence; ensure no grid/fixed introduced.
3) Homepage limit + Archive page
   - Change: `index.md` shows last 100; new `archive.md` lists all (same markup); keep all posts online for SEO.
   - Tests: assert Liquid `limit:` exists; archive includes anchors.
4) Feed smoke: at least one <entry>
   - Change: add curl/grep to `rss-smoke.yml`.
   - Tests: optional unit asserting workflow text contains the grep.
5) Tiny perceived-speed bump
   - Change: add `<link rel="prefetch" href="{{ '/assets/search-index.json' | relative_url }}">` on home only.
   - Tests: assert tag present in layout.

Oct 26, 2025 — Content Width Cap
- Change: added `section { max-width: 900px; }` in `docs/assets/css/site.css` to prevent overly wide content and reduce rare overlap with the left header on very wide viewports.
- Test: `tests/test_layout_content_width.py` asserts the presence of the cap.
- Rationale: minimal, no layout restructuring, keeps the theme’s default margins.

Oct 26, 2025 — Search Edge-Case Tests
- Added `tests/test_search_edge_cases_strings.py` to lock in behaviors:
  - Empty query clears results and returns early.
  - Highlight escapes regex characters and uses <mark>.
  - Multi-token split loop present; tokens length>1 to highlight.
  - Enter does nothing when no results (guard present).
- Relaxed an over-strict layout assertion to allow the single absolute-position used by the dropdown.
Overlap fix — Oct 26, 2025
- Removed `.wrapper { display:flex; ... }` and related `section/footer` flex rules in `site.css` to restore theme layout and prevent wide-screen overlap with the left header.
- Tests: `tests/test_layout_no_flex_wrapper.py` ensures we don't reintroduce a flex wrapper; relaxed `test_layout_basic.py` to not require a `section { ... }` rule in custom CSS.
CI concurrency — Oct 26, 2025
- Added workflow `concurrency` to Deploy Jekyll site:
  ```yaml
  concurrency:
    group: pages-${{ github.ref }}
    cancel-in-progress: true
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tom-doerr/repo_posts](https://github.com/tom-doerr/repo_posts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
