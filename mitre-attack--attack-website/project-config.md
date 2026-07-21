---
trigger: always_on
description: This file is guidance for coding agents working in `attack-website`.
---

# AGENTS.md

This file is guidance for coding agents working in `attack-website`.

## Scope

- Applies to the repository root.
- There was no existing `AGENTS.md` to preserve.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.
- Primary human docs are `DEVELOPMENT.md`, `README.md`, `test/README.md`, and `CONTRIBUTING.md`.

## Repo Shape

- Root Python build system generates the static site via `update-attack.py`.
- `attack-search/` is a separate Node/CommonJS project for the search bundle.
- Search source and tests live in `attack-search/src/` and `attack-search/__tests__/`.
- `attack-style/` is a separate Node/Sass project for CSS output.
- SCSS entrypoints are `attack-style/style-attack.scss` and `attack-style/style-user.scss`.
- `attack-theme/` contains Jinja templates, static assets, and legacy browser JS.
- Theme templates and static assets live in `attack-theme/templates/` and `attack-theme/static/`.
- `modules/` contains Python modules that generate ATT&CK site content.
- `test/` provides an Nginx Docker environment for validating the built site.

## Environment Expectations

- Python 3 is required for the main build.
- When managing a local Python environment, prefer `uv` with a virtual environment at `.venv` in the git repository root.
- Node.js and npm are required for `attack-search/` and `attack-style/`.
- Docker is the preferred way to validate the final static output in an Nginx-like environment.
- CI currently uses Python `3.13` and Node `18.x` in `.github/workflows/gh-pages.yml`.
- Prefer CI versions when reproducing CI behavior; Docker and development docs may reference older base images.
- Production-like builds may depend on environment variables from `.github/workflows/gh-pages.yml`, including `GOOGLE_ANALYTICS`, `GOOGLE_SITE_VERIFICATION`, `INCLUDE_OSANO`, and `PELICAN_SITEURL`.

## High-Value Commands

Run commands from the repo root unless a subdirectory is called out.

### Install

- Preferred Python env: `uv venv .venv`
- Python deps: `uv pip install -r requirements.txt`
- Search deps: `cd attack-search && npm ci`
- Style deps: `cd attack-style && npm ci`
- Prefer `npm ci` over `npm install`; do not update lockfiles unless dependency changes are part of the task.

### Build

- Main website build: `uv run python update-attack.py --attack-brand --extras --no-test-exitstatus`
- Search bundle: `cd attack-search && npm run build`
- Search dev bundle: `cd attack-search && npm run build:dev`
- Copy built search bundle into site output: `cd attack-search && npm run copy`
- Style build: `cd attack-style && npm run build`
- Style build + copy into theme static assets: `cd attack-style && npm run build-copy`

### Local Validation

- Full local site validation follows `DEVELOPMENT.md` and `test/README.md`.
- Build site output first, then serve `output/` through the Docker test image.
- Test container build: `cd test && docker build -t attack-website-test .`
- Test container run: `cd test && docker run -p 80:80 -v $(pwd)/../output:/workspace attack-website-test`
- Helper script: `cd test && ./run_test.sh`

### Lint And Format

- Python lint (configured, not wired into CI): `ruff check .`
- Python lint autofix: `ruff check --fix .`
- Python format: `ruff format .`
- Search lint: `cd attack-search && npm run lint`
- Search lint autofix: `cd attack-search && npm run lint:fix`
- Style lint: `cd attack-style && npm run lint`

### Tests

- Search tests: `cd attack-search && npm test`
- Single Jest test file: `cd attack-search && npm test -- __tests__/search-service.test.js`
- Alternate single Jest file: `cd attack-search && npx jest __tests__/search-service.test.js`
- Main Python-driven site tests run through the build script, not `pytest`.
- Run specific site test categories: `uv run python update-attack.py -m tests -t size`
- Multiple site test categories: `uv run python update-attack.py -m tests -t links external_links citations`

### Important Command Notes

- There is no root `package.json`, `Makefile`, or single universal test runner.
- CI clearly builds the site and search bundle, but does not currently enforce Jest, ESLint, Stylelint, Ruff, or type checks.
- For Python-side testing, the narrowest supported scope is a named category (`size`, `links`, `external_links`, `citations`), not an individual test file.
- Preferred production-like validation is Nginx via Docker, not Pelican's built-in dev server.
- Pelican's built-in development server does not match production Nginx routing behavior.

## Source Of Truth

- Follow existing file-local conventions before applying generic preferences.
- Treat `pyproject.toml`, `attack-search/.eslintrc`, and `attack-style/.stylelintrc.json` as authoritative style configs.
- Treat `DEVELOPMENT.md` and `.github/workflows/gh-pages.yml` as authoritative for build workflow.
- In templates, respect comments that mark generated files or source-of-truth files.
- Example: `attack-theme/templates/general/base-template.html` explicitly says to edit `base-template.html`, not generated `base.html`.

## Generated And Volatile Outputs

- Do not edit `output/` as source; regenerate it through the build pipeline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitre-attack/attack-website](https://github.com/mitre-attack/attack-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
