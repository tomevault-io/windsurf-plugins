---
trigger: always_on
description: Orientation for agents (Claude, Copilot, Codex, and others) working in this repo. A short install/setup summary lives in [README.md](README.md); the full user and technical documentation lives in `docs-site/` and is published to GitHub Pages at <https://appz4fun.github.io/NeNeTeePee-Stream-Kodi/>. Outstanding work lives in [TODO.md](TODO.md).
---

# AGENTS.md

Orientation for agents (Claude, Copilot, Codex, and others) working in this repo. A short install/setup summary lives in [README.md](README.md); the full user and technical documentation lives in `docs-site/` and is published to GitHub Pages at <https://appz4fun.github.io/NeNeTeePee-Stream-Kodi/>. Outstanding work lives in [TODO.md](TODO.md).

## Summary

- Runtime addon code must stay Python 3.8 compatible and pure Python.
- Preserve `setResolvedUrl` (resolvable plugin routes), `waitForAbort`, and HTTP Range behavior.
- Follow existing Kodi mock, settings, HTTP helper, and player install patterns.
- Run `just lint` and `just test` before commit or push.
- For releases, bump only `repo/plugin.video.nzbdav/addon.xml`; the Release workflow builds the zip and the external Appz4Fun Kodi repository republishes it. The Pages workflow publishes documentation, not add-on metadata.

## Agent contract

Follow these rules before making code, release, or deployment changes:

- Run `just lint` and `just test` before any `git commit` or `git push`.
- If `just lint` reports formatting issues, run `just lint-fix`, then re-run `just lint`.
- Keep runtime addon code Python 3.8 compatible. No walrus operators, `match`, or `str.removeprefix`.
- Do not add compiled dependencies or C extensions. CoreELEC/ARM64 installs must stay pure Python.
- Do not edit vendored PTT under `repo/plugin.video.nzbdav/resources/lib/ptt/` unless fixing compatibility.
- Do not duplicate shared HTTP or notification helpers; use `http_util.py`.
- Only bump `repo/plugin.video.nzbdav/addon.xml` for add-on releases. This repo no longer contains a Kodi repository add-on.
- Do not hand-edit the generated docs site under `site/`. The documentation source lives in `docs-site/` and is published to GitHub Pages by the Docs workflow.
- Do not commit real API keys, WebDAV credentials, Kodi logs, copied crash logs, or local device artifacts.

## Critical invariants

These must stay true or Kodi playback, shutdown, or updates can break:

- Every resolvable plugin route (`/play`, `/direct_play`, and `resolver.resolve(handle, ...)`) must call `xbmcplugin.setResolvedUrl(...)` on every path: success with `True`, failure or cancellation with `False`.
- Handle-less playback paths have no plugin handle. They are the TMDBHelper `RunScript(addon.py,tmdb_play,...)` player, `/resolve`, `/resolve-v2`, `/search` (after `endOfDirectory`), and `resolve_and_play`.
- Handle-less paths start playback with `xbmc.Player().play(...)` instead of `setResolvedUrl`.
- On failure, handle-less paths notify the user.
- Handle-less failure cleanup closes open progress dialogs.
- Handle-less failure cleanup also stops background fallback workers.
- If a change deliberately revises that failure handling, update the tests to match.
- The NZBGet backend supports both kinds of entry path.
- Kodi polling loops must use `xbmc.Monitor.waitForAbort()` instead of `time.sleep()` so Kodi can shut down cleanly.
- Settings must be defined in `resources/settings.xml` and read through `xbmcaddon.Addon().getSetting(...)`.
- The stream proxy must preserve HTTP Range behavior; seeking depends on it.
- MP4 rewrite and ffmpeg remux paths must keep ffmpeg optional and degrade gracefully when it is missing.
- Non-MP4 pass-through is the default unless settings explicitly choose a force-remux path.
- Test imports depend on `tests/conftest.py` pre-mocking `xbmc*` modules before `resources.lib.*` imports.

## Fast commands

```bash
just test          # Run all tests
just lint          # ruff + black + pylint + vermin
just lint-fix      # Auto-fix lint/format issues, then re-run just lint
just ci            # Same checks as GitHub CI: lint + test + Python 3.8 compileall gate
just release       # Build plugin.video.nzbdav-<version>.zip
just ship          # test + release
just deploy-addon  # Push the addon tree to the CoreELEC box and restart Kodi
just version       # Print the current addon version from addon.xml
just changelog     # Show the Kodi-visible addon changelog
just docs          # Build the MkDocs docs site into ./site (strict)
just docs-serve    # Serve the docs site locally with live reload
just extreme-tests # Run the extreme end-to-end fault-recovery test
just clean         # Remove __pycache__, .pytest_cache, zip
just dist-clean    # clean + remove the generated docs site
```

## PR review helper scripts

For agent PR review workflows:

- `python3 scripts/pr_agent_context.py --json` -- preferred unified agent context packet.
- `python3 scripts/pr_review_context.py --json` -- local branch, PR, check, and file context.
- `python3 scripts/fetch_comments.py --json` -- unresolved GitHub review threads and PR comments.

Use `fetch_comments.py` directly when a skill or workflow expects that helper by name.
Use `pr_agent_context.py` when starting a PR review or addressing comments from scratch.

## Repository map

- `repo/plugin.video.nzbdav/` -- Kodi addon installed via zip

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Appz4Fun/NeNeTeePee-Stream-Kodi](https://github.com/Appz4Fun/NeNeTeePee-Stream-Kodi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
