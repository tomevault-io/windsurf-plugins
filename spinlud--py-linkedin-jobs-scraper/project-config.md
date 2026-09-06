---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`linkedin-jobs-scraper`: a PyPI package that scrapes public LinkedIn job postings by driving a headless Chrome instance through Selenium. There is a sibling npm package (`linkedin-jobs-scraper`) with equivalent behaviour — feature parity between the two is intentional.

## Commands

The project uses [uv](https://docs.astral.sh/uv/) for the environment and packaging; `pyproject.toml` (hatchling backend, PEP 621) declares dependencies and metadata, `.python-version` pins the local interpreter to 3.13, and `uv.lock` is committed. `uv run` and `uv sync` provision the env from those on demand, so there is no separate install step. The `package.json` scripts are the canonical entry points and wrap uv:

```shell
npm run hooks    # git config core.hooksPath .githooks (once per clone)
npm run test     # uv run pytest tests/unit --capture=no --log-cli-level=DEBUG (offline unit suite)
npm run clean    # remove build/, dist/, *.egg-info, __pycache__, .pytest_cache
npm run build    # clean + uv build (sdist + wheel into dist/)
npm run deploy   # uv publish to testpypi

npm run test:e2e:programmatic   # live programmatic suite (needs credentials)
npm run test:e2e:cli            # live CLI suite (needs credentials)
npm run test:e2e                # both live suites
```

`.githooks/pre-commit` bumps the patch version in `pyproject.toml` and stages it, so every commit carries its own version. Git never enables a cloned repository's hooks on its own, so `npm run hooks` (or the `git config` behind it) is needed once per clone; `git commit --no-verify` skips it for a single commit. It stands down while git is replaying commits that already carry a version — merge, rebase, cherry-pick, revert — and when `pyproject.toml` holds unstaged changes, which staging the bump would sweep into the commit.

The cost is structural, not a defect: every commit touches the same line, so branches that diverge collide on it, and a cherry-pick across them stops on a conflict in `pyproject.toml`. Versions also count commits rather than releases. Bumping on push to `master`, or deriving the version from a git tag, are the two ways out if that becomes tiresome.

Running the live suites directly (CI runs only the offline unit tests under `tests/unit/`; the live suites below are run locally before merging to `master`). The `npm run test:e2e:*` scripts above wrap these; run them directly to pass a credential inline:

```shell
LI_RM_COOKIE=<li_rm> LI_BCOOKIE=<bcookie> uv run pytest tests/test_programmatic.py::test_run   # programmatic live suite
LI_RM_COOKIE=<li_rm> LI_BCOOKIE=<bcookie> uv run pytest tests/test_cli.py                       # CLI live suite
```

`LI_CHROME_USER_DATA_DIR=<path>` is an alternative to the cookie pair for both live suites: point a run at a seeded Chrome profile that already holds a session (the `login` subcommand produces one) and no env cookies are needed. The suites pass it through — to `LinkedinScraper(chrome_user_data_dir=...)` and to the CLI's `--chrome-user-data-dir` — and the "profile wins" logic authenticates from the profile's own session.

Selenium Manager fetches a chromedriver matching the local Chrome, so there is nothing to install — but it will not override a mismatched chromedriver already on `PATH`, so locally `PATH="/usr/bin:/bin"` is the way to keep it out of the way.

**The live suites hit the real LinkedIn site.** Two of them, run locally rather than in CI: `tests/test_programmatic.py` drives the library in-process (`tests/shared.py` asserts on the shape of each emitted `EventData`), and `tests/test_cli.py` runs the CLI as a subprocess and asserts on its exit codes and jsonl stdout. Both need no fixtures and take a credential from the environment — use the remember me pair rather than `LI_AT_COOKIE`, which each suite exhausts in about two runs. A failing live test usually means LinkedIn changed its DOM (see *Selectors*), not that the Python logic broke. Offline unit tests live under `tests/unit/` and need no credential.

`tests/manual/` holds standalone probes that need no live LinkedIn: `throttle_backoff.py` (backoff ladder and pacer, against a local server), `mid_run_recovery.py`, `remote_probe.py`, `network_headers_probe.py`, `validate_fields.py`.

Release: pushing to `master` publishes to PyPI via `.github/workflows/ci.yml`. Version is declared only in `pyproject.toml` (`package.json`'s version is unused). Because a push to `master` publishes, never push there without the maintainer explicitly asking.

## Architecture

Flow: `LinkedinScraper.run(queries)` → one `ThreadPoolExecutor` task per `Query` → **one Chrome driver per task**, reused across the loop over `query.options.locations`, delegating to `AuthenticatedStrategy`. One browser per query rather than per location, because every new browser is another session establishment for LinkedIn to look at.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spinlud/py-linkedin-jobs-scraper](https://github.com/spinlud/py-linkedin-jobs-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
