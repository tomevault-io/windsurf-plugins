---
trigger: always_on
description: - `ytmusicfs` is a Python FUSE filesystem for browsing and streaming YouTube Music as local files.
---

# Repository Guidelines

## Project Overview

- `ytmusicfs` is a Python FUSE filesystem for browsing and streaming YouTube Music as local files.
- It captures YouTube Music browser auth, reads library data through `ytmusicapi`, and streams audio through `yt-dlp`.
- Keep behavior practical for media players: stable paths, predictable metadata, cached listings, and graceful failures.

## Approach

- Read before editing. Test before declaring done.
- Prefer small edits over rewrites.
- Reproduce before fixing runtime or external issues.
- Unproven concerns are risks, not bugs. Say so if not reproduced.
- Use the simplest working solution. No over-engineering, speculative features, or single-use abstractions.

## Output

- Code first. Explain only non-obvious logic.
- No filler, boilerplate, or out-of-scope suggestions.
- Use plain hyphens and straight quotes only. No decorative Unicode. Keep code output copy-paste safe.

## Project Structure & Modules

- `ytmusicfs/`: Core package.
  - `cli.py`: CLI entry point, argument parsing, logging, mount orchestration.
  - `filesystem.py`: Main FUSE implementation and filesystem operations.
  - `client.py`: YouTube Music API wrapper.
  - `auth_adapter.py`: Browser cookie auth for `ytmusicapi.YTMusic`.
  - `content_fetcher.py`: Library retrieval, playlist registry, cache coordination.
  - `processor.py`: Filename sanitizing, path/video ID extraction, track shaping.
  - `cache.py`: SQLite persistent cache plus in-memory caches.
  - `file_handler.py`, `downloader.py`, `yt_dlp_utils.py`: File handles and audio stream extraction.
  - `repair.py`: Liked-song repair flow for unavailable video IDs and account rating updates.
  - `path_router.py`: Filesystem path routing and validation.
  - `metadata.py`: Track metadata mapping and cache integration.
  - `thread_manager.py`: Thread pools and concurrent work helpers.
  - `config.py`: Default paths for auth, cache, and logs.
- `tests/`: Pytest suite for cache, routing, filesystem, downloader, and integration behavior.
- Root configs: `pyproject.toml`, `pytest.ini`, `.editorconfig`.
- Extras: `.github/`, `.devcontainer/`, `build/`.

## Build, Test, and Development Commands

- Install local app: `pipx install .`
- Refresh local app after changes: `pipx install --force .`
- Check CLI: `ytmusicfs --version`
- Mount: `ytmusicfs mount --mount-point ~/Music/ytmusic --browser brave`
- Mount with saved settings after first successful mount: `ytmusicfs mount`
- Debug mount with browser cookies: `ytmusicfs mount --mount-point ~/Music/ytmusic --browser brave --foreground --debug`
- Unmount active mount: `ytmusicfs unmount`
- Unmount explicit path: `ytmusicfs unmount --mount-point ~/Music/ytmusic`
- Status: `ytmusicfs status`
- Doctor: `ytmusicfs doctor`
- Saved config: `ytmusicfs config show`, `ytmusicfs config set browser brave`, `ytmusicfs config set mount-point ~/Music/ytmusic`
- Cache: `ytmusicfs cache stats`, `ytmusicfs cache clear` (metadata and audio), `ytmusicfs cache refresh` (metadata only)
- Repair unavailable liked-song IDs: `ytmusicfs repair`
- Logs: `ytmusicfs logs` (last 50 lines), `ytmusicfs logs --tail N`, `ytmusicfs logs --path`
- Systemd user service: `ytmusicfs service install`, `ytmusicfs service start`, `ytmusicfs service stop`
- Mounted debug status: `/.ytmusicfs/status.json`.
- Tests: `pipx run --spec '.[dev]' pytest -q`
- Verbose tests: `pipx run --spec '.[dev]' pytest -v`
- Focused tests: `pipx run --spec '.[dev]' pytest -k path_router -q`
- Coverage: `pipx run --spec '.[dev]' pytest --cov=ytmusicfs`
- Lint/type/format check: `pipx run --spec '.[dev]' black --check . && pipx run --spec '.[dev]' ruff check . && pipx run --spec '.[dev]' mypy ytmusicfs`
- Auto-format: `pipx run --spec '.[dev]' black . && pipx run --spec '.[dev]' ruff check --fix .`
- Build wheel/sdist: `pipx run --spec build pyproject-build`

## Coding Style & Naming

- Python 3.10+ project; prefer type hints and explicit return types.
- Formatter: Black, line length 88.
- Imports: Ruff import sorting (`I` rules), formatted with Black.
- Indentation: 4 spaces.
- Names: modules/functions `snake_case`, classes `PascalCase`, constants `UPPER_SNAKE_CASE`.
- Use `logging` instead of prints. Follow `ytmusicfs/cli.py` logging setup.
- Keep functions focused. Prefer small component-level changes over cross-cutting rewrites.
- Remove unused imports, variables, parameters, dead branches, and dead functions from edited files.
- Do not add error handling for impossible scenarios.
- Keep all imports at the top of the file. Use local imports only when strictly required to break circular dependencies.
- Code and comments must be in English. User-facing strings stay in their existing language.
- Remove old code when introducing replacements. Do not add backward compatibility shims without explicit authorization.
- Do not preserve feature flags for shipped features or abstractions that serve a single caller.

## Architecture Notes

- Components communicate through narrow interfaces; avoid reaching into another component's internals.
- `YouTubeMusicFS` coordinates routing, metadata, content fetches, cache, and file handling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astrovm/ytmusicfs](https://github.com/astrovm/ytmusicfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
