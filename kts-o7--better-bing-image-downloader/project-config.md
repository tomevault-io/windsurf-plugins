---
trigger: always_on
description: Notes for AI coding agents (Claude, GPT, Cursor, etc.) working in this
---

# AGENTS.md

Notes for AI coding agents (Claude, GPT, Cursor, etc.) working in this
repository. Read this before making non-trivial changes.

## Project at a glance

- **What it is:** Python library + `bbid` CLI for bulk-downloading
  images from Bing or DuckDuckGo. The recommended way to use it as a
  library is the `Downloader` class (added in v3.2.0).
- **Python:** 3.8+. Pure-stdlib HTTP (urllib), `brotli` for DuckDuckGo
  (hard dep since v3.1.1), optional `selenium` for the legacy
  `multidownloader` path.
- **No CI is configured** (per the maintainer's request). Lint/type/test
  is run locally before pushing.
- **Tests:** 149 passing, 2 network tests skipped by default
  (`BBID_RUN_NETWORK_TESTS=1` to enable).
- **Linters:** `black` (formatter), `ruff` (lint), `mypy` (types).
  All three run via pre-commit.

## File map

| File | What it does | Modify freely? |
|------|--------------|----------------|
| `base.py` | `ImageEngine` abstract base class — atomic write, dedup, resume, manifest, parallel downloads, future timeout. Defines `run()` as `@abstractmethod` | ✅ Yes |
| `bing.py` | Bing image search engine (inherits from `base.ImageEngine`) | ✅ Yes |
| `duckduckgo.py` | DuckDuckGo image search engine (inherits from `base.ImageEngine`) | ✅ Yes |
| `manifest.py` | `ManifestWriter` JSONL writer (v3.5.0+). Reusable independently of `Downloader` | ✅ Yes |
| `downloader.py` | `Downloader` class, engine registry, lifecycle hooks. **v3.2.0+ entry point** | ✅ Yes |
| `results.py` | `Result` and `ImageResult` value objects returned by `Downloader.search()` | ✅ Yes |
| `download.py` | Module-level `downloader()` function (legacy wrapper around `Downloader`) and `bbid` CLI | ✅ Yes |
| `__init__.py` | Public API surface — re-exports `Bing`, `DuckDuckGo`, `Downloader`, `ImageResult`, `Result`, `downloader` | ✅ Yes |
| `crawler.py` | **DEPRECATED** Selenium-based crawler | ⚠️ Don't extend; remove in v4.0.0 |
| `multidownloader.py` | **DEPRECATED** Selenium-based CLI | ⚠️ Don't extend; remove in v4.0.0 |
| `helperdownload.py` | **DEPRECATED** Used by `multidownloader` | ⚠️ Don't extend; remove in v4.0.0 |
| `utils.py` | **DEPRECATED** Config helpers | ⚠️ Don't extend; remove in v4.0.0 |
| `pyproject.toml` | Package metadata, version, tool config | ✅ Yes (bump version on release) |
| `requirements*.txt` | Mirror of `[project.optional-dependencies]` | ✅ Yes |
| `tests/` | pytest test suite | ✅ Yes |
| `README.md` | User docs | ✅ Yes |
| `CHANGELOG.md` | Release notes | ✅ Yes (add an "Unreleased" section) |
| `CONTRIBUTING.md` | Contributor guide | ✅ Yes |
| `.pre-commit-config.yaml` | Pre-commit hook config | ✅ Yes (pin revisions carefully) |

## Patterns and conventions

### Adding a new feature

1. **Read `base.py` first.** Most download-side concerns are already
   handled there. New features should reuse `download_image()` and
   `_download_batch()` instead of duplicating the parallel-download
   logic.
2. **TDD where possible.** The existing tests mock at module-attribute
   boundaries (`urllib.request.urlopen`, `filetype.guess`,
   `requests.get`). Follow the pattern in `tests/test_duckduckgo.py`.
3. **No new top-level dependencies** without discussion. The package
   has stayed stdlib-only for the Bing path; optional extras are
   `[duckduckgo]` (brotli) and `[google]` (selenium).

### Test patterns

- Tests use `pytest` (not unittest, except `tests/test_bing.py` which
  was written in unittest style — both work, prefer pytest for new
  tests).
- Patches target the **module that uses** the symbol, not the symbol's
  origin. Example: `filetype` is imported in `base.py`, so patch
  `better_bing_image_downloader.base.filetype.guess`, not
  `better_bing_image_downloader.bing.filetype.guess`.
- Network tests must be gated on `BBID_RUN_NETWORK_TESTS=1`.
- The end-to-end DDG test is the only network test today. Add new
  end-to-end tests behind the same gate.

### Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/):
`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`. Scope is
optional but useful (`feat(duckduckgo): add region code`).

### Versioning

- **Patch** (3.1.x): bug fixes, no API change.
- **Minor** (3.x.0): new features, backwards-compatible.
- **Major** (x.0.0): breaking changes.

The current version is **3.5.1**. Bump in `pyproject.toml` and
`CHANGELOG.md` when cutting a release. Releases are cut by:
1. Committing on `main`.
2. Tagging (`git tag -a v3.X.Y -m "v3.X.Y: summary"`).
3. Pushing the tag.
4. Creating a GitHub release — the `python-publish.yml` workflow
   publishes to PyPI automatically.

### Code style

- **Line length:** 100 (enforced by black + ruff).
- **Type hints:** required on public API. `base.py`, `bing.py`,
  `duckduckgo.py`, and `download.py` are typed; deprecated modules
  are not.
- **Docstrings:** Google-style or NumPy-style; the existing code mixes
  both. Match the style of the file you're editing.
- **Logging:** use `logging`, never `print()`. The `helperdownload`
  migration to `logging` was a deliberate fix; don't reintroduce
  `print()` calls.

## What NOT to do

- **Don't reintroduce `print()` in library code.** It's a library; users
  may pipe stdout.
- **Don't add Selenium or other browser-automation deps to the core.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KTS-o7/better_bing_image_downloader](https://github.com/KTS-o7/better_bing_image_downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
