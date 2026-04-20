---
trigger: always_on
description: Cross-tool agent reference for the Houndarr repository.
---

# AGENTS.md: Houndarr

Cross-tool agent reference for the Houndarr repository.
This file is the primary source of truth for autonomous agents operating here.

## Project Overview

Houndarr is a self-hosted companion for Radarr, Sonarr, Lidarr, Readarr, and
Whisparr that automatically searches for missing, cutoff-unmet, and
upgrade-eligible media in small, rate-limited batches. It runs as a single Docker container alongside
an existing *arr stack.

**Tech stack:** Python 3.12 / FastAPI / aiosqlite (SQLite) / Jinja2 / HTMX /
Tailwind CSS CDN. Published to GHCR at `ghcr.io/av1155/houndarr`.

**Scope guard:** Houndarr is a single-purpose tool. Every change must help
search for missing, cutoff-unmet, or upgrade-eligible media in a controlled,
polite way.
Do not add download-client integration, indexer management, request workflows,
multi-user support, or media file manipulation.

---

## Setup & Run

```bash
# Create venv and install
python3 -m venv .venv
.venv/bin/pip install --upgrade pip
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/pip install -e .

# Run locally (dev mode; auto-reload, API docs at /docs)
.venv/bin/python -m houndarr --data-dir ./data-dev --dev
```

Dev server: `http://localhost:8877`.

---

## Quality Gates

Run **all five** before every commit. These are the core local gates; CI
also enforces them as part of the required checks, alongside additional
security and container checks.

```bash
.venv/bin/python -m ruff check src/ tests/          # lint
.venv/bin/python -m ruff format --check src/ tests/  # format check
.venv/bin/python -m mypy src/                        # type check (strict)
.venv/bin/python -m bandit -r src/ -c pyproject.toml # SAST
.venv/bin/pytest                                     # all tests
```

---

## Running Tests

```bash
# Full suite (949 tests, async; count includes parametrised expansions)
.venv/bin/pytest

# Single file
.venv/bin/pytest tests/test_auth.py

# Single test by name
.venv/bin/pytest tests/test_auth.py::test_check_password_valid -v

# Tests matching a keyword expression
.venv/bin/pytest -k "csrf" -v

# Single directory
.venv/bin/pytest tests/test_services/

# With coverage
.venv/bin/pytest --cov=houndarr --cov-report=term-missing
```

**Pytest config** (from `pyproject.toml`):

- `asyncio_mode = "auto"`: async tests run without manual event-loop setup
- `asyncio_default_fixture_loop_scope = "function"`: each test gets its own loop
- `addopts = "-q --tb=short"`: default quiet output with short tracebacks

---

## CI Checks

### Required checks (11; branch protection enforced)

| Check name | Workflow file | What it runs |
|------------|---------------|--------------|
| Lint (ruff) | `quality.yml` | `ruff check .` |
| Format (ruff) | `quality.yml` | `ruff format --check .` |
| Type check (mypy) | `quality.yml` | `mypy src/` |
| Test (Python 3.12) | `tests.yml` | `pytest -q --tb=short` + compile check + `--help` |
| Dependency audit (pip-audit) | `security.yml` | `pip-audit -r requirements.txt -r requirements-dev.txt` |
| SAST (bandit) | `security.yml` | `bandit -r src/ -c pyproject.toml` |
| Trivy filesystem scan | `security.yml` | `trivy fs .` (CRITICAL/HIGH with known fix) |
| Dependency review | `dependency-review.yml` | PR dependency diff vs GitHub Advisory Database |
| Build (no push) | `docker.yml` | Multi-arch Docker build (amd64/arm64), no push |
| Trivy image scan | `docker.yml` | Trivy scan of built Docker image (CRITICAL/HIGH with known fix) |
| Security smoke test | `security-smoke-test.yml` | Live container: unauthenticated sweep, CSRF, XFF, rate limiting, API key exposure, container security |

The six main workflows (`quality`, `tests`, `security`, `dependency-review`,
`docker`, `security-smoke-test`) use `paths-ignore: ["docs/**", "**/*.md", "website/**", ".claude/**"]`. When a PR
touches only those paths, `ci-skip.yml` provides passing no-op jobs with
identical check names so branch protection is satisfied.

### Additional workflows (not required checks)

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `version-check.yml` | PRs changing `VERSION` or `CHANGELOG.md` | Validates VERSION format, CHANGELOG heading match, allowed `###` headers, `---` separator |
| `release.yml` | `v*` tag push | Validates VERSION == tag, extracts CHANGELOG block, creates GitHub Release |
| `chart.yml` | `v*` tag push | Packages `charts/houndarr/` with version from `VERSION` file, pushes to `oci://ghcr.io/av1155/charts` |
| `dockerfile-lint.yml` | Changes to `Dockerfile` | `hadolint Dockerfile` |
| `workflow-lint.yml` | Changes to `.github/workflows/**` | `actionlint` via reviewdog |
| `api-snapshot-refresh.yml` | Weekly (Monday 10:00 UTC) + manual | Fetches upstream Radarr/Sonarr/Whisparr/Lidarr/Readarr OpenAPI specs, updates `docs/api/` snapshots and `tests/test_docs_api.py` hashes, opens a PR if changed |
| `pages.yml` | Pushes to `main` touching `website/**` | Deploys docs site to GitHub Pages |
| `test-deploy.yml` | PRs touching `website/**` | Tests Docusaurus build without deploying |
| `link-check.yml` | PRs touching `**/*.md`, `**/*.mdx`, `lychee.toml` + weekly (Monday 08:00 UTC) + manual | Runs `lychee` against every Markdown file to catch broken external links; rules live in `lychee.toml` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [av1155/houndarr](https://github.com/av1155/houndarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
