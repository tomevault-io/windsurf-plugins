---
trigger: always_on
description: - `app/` holds core logic, configuration, and tools (e.g., `app/tool/meetspot_recommender.py` and the in-progress `design_tokens.py`). Treat it as the authoritative source for business rules.
---

# Repository Guidelines

## Project Structure & Module Organization
- `app/` holds core logic, configuration, and tools (e.g., `app/tool/meetspot_recommender.py` and the in-progress `design_tokens.py`). Treat it as the authoritative source for business rules.
- `api/index.py` wires FastAPI, middleware, and routers; `web_server.py` bootstraps the same app locally or in production.
- Presentation assets live in `templates/` (Jinja UI), `static/` (CSS, icons), and `public/` (standalone marketing pages); generated HTML drops under `workspace/js_src/` and should be commit-free.
- Configuration samples sit in `config/`, docs in `docs/`, and regression or SEO tooling in `tests/` plus future `tools/` automation scripts.

## Build, Test, and Development Commands
- `pip install -r requirements.txt` (or `conda env update -f environment-dev.yml`) installs Python 3.11 dependencies.
- `python web_server.py` starts the full stack with auto env detection; `uvicorn api.index:app --reload` is preferred while iterating.
- `npm run dev` / `npm start` proxy to the same Python entry point for platforms that expect Node scripts.
- `pytest tests/ -v` runs the suite; `pytest --cov=app tests/` enforces coverage; `python tests/test_seo.py http://127.0.0.1:8000` performs the SEO audit once the server is live.
- Quality gates: `black .`, `ruff check .`, and `mypy app/` must be clean before opening a PR.

## Coding Style & Naming Conventions
- Python: 4-space indent, type hints everywhere, `snake_case` for functions, `PascalCase` for classes, and `SCREAMING_SNAKE_CASE` for constants. Keep functions under ~50 lines and prefer dataclasses for structured payloads.
- HTML/CSS: prefer BEM-like class names (`meetspot-header__title`), declare shared colors via the upcoming `static/css/design-tokens.css`, and keep inline styles limited to offline-only HTML in `workspace/js_src/`.
- Logging flows through `app/logger.py`; use structured messages (`logger.info("geo_center_calculated", extra={...})`) so log parsing stays reliable.

## Testing Guidelines
- Place new tests in `tests/` using `test_<feature>.py` naming; target fixtures that hit both FastAPI routes and tool-layer helpers.
- Maintain ≥80% coverage for the `app/` package; add focused tests when touching caching, concurrency, or SEO logic.
- Integration checks: run `python tests/test_seo.py <base_url>` against a live server and capture JSON output in the PR for visibility.
- Planned accessibility tooling (`tests/test_accessibility.py`) will be part of CI—mirror its structure for any lint-like tests you add.

## Commit & Pull Request Guidelines
- Follow Conventional Commits (`feat:`, `fix:`, `ci:`, `docs:`) as seen in `git log`; keep scopes small (e.g., `feat(tokens): add WCAG palette`).
- Reference related issues in the first line of the PR description, include a summary of user impact, and attach screenshots/GIFs for UI work.
- List the commands/tests you ran, note any config changes (e.g., `config/config.toml`), and mention follow-up tasks when applicable.
- Avoid committing generated artifacts from `workspace/` or credentials in `config/config.toml`; add new secrets to `.env` or deployment config.

## Configuration & Architecture Notes
- Keep `config/config.toml.example` updated when introducing new settings, and never hardcode API keys—read them via `app.config`.
- The design-token and accessibility architecture is tracked in `.claude/specs/improve-ui-ux-color-scheme/02-system-architecture.md`; align contributions with that spec and document deviations in your PR.


## Skills
A skill is a set of local instructions to follow that is stored in a `SKILL.md` file. Below is the list of skills that can be used. Each entry includes a name, description, and file path so you can open the source for full instructions when using a specific skill.
### Available skills
- find-skills: Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. This skill should be used when the user is looking for functionality that might exist as an installable skill. (file: /Users/calder/.agents/skills/find-skills/SKILL.md)
- gh-address-comments: Help address review/issue comments on the open GitHub PR for the current branch using gh CLI; verify gh auth first and prompt the user to authenticate if not logged in. (file: /Users/calder/.codex/skills/gh-address-comments/SKILL.md)
- gh-fix-ci: Use when a user asks to debug or fix failing GitHub PR checks that run in GitHub Actions; use `gh` to inspect checks and logs, summarize failure context, draft a fix plan, and implement only after explicit approval. Treat external providers (for example Buildkite) as out of scope and report only the details URL. (file: /Users/calder/.codex/skills/gh-fix-ci/SKILL.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calderbuild/MeetSpot](https://github.com/calderbuild/MeetSpot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
