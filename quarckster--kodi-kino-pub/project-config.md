---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Kodi 20+ video add-on for the kino.pub online movie service (the 4.x.x series supports Kodi 20, 21,
and 22; for Kodi 19 use 3.x.x). The runtime is Kodi's embedded Python: modules like `xbmc`,
`xbmcaddon`, `xbmcgui`, `xbmcplugin`, `xbmcvfs` are provided by Kodi itself (stubbed via `Kodistubs`
for local dev / type-checking, not real pip packages). Add-on source under `src/` must stay **Python
3.8 compatible**; dev tooling and tests run on Python 3.10. Video metadata is set through the
`InfoTagVideo` setters (Kodi 20+), not the deprecated `ListItem.setInfo()`.

## Commands

Setup: install [`uv`](https://docs.astral.sh/uv/), then `uv sync` to create `.venv` and install the
dev tooling (and install [`podman`](https://podman.io) for integration tests). Dependencies are
locked in `uv.lock` (dev group in `pyproject.toml`); bump them with `uv lock --upgrade`. Prefix the
commands below with `uv run` (e.g. `uv run make test_unit`) or activate `.venv` first.

- **Lint / format / type-check:** `uv run pre-commit run --all` (black @ line-length 100, flake8,
  mypy, reorder-python-imports, pyupgrade).
- **Unit tests:** `make test_unit` → `pytest -v -m "not integration"`. These mock `xbmc*` and need no
  containers.
- **Integration tests:** `make test_integration` → `pytest -v -m integration`. Requires podman;
  spins up real Kodi + a mock API server (see Testing below). CI runs these against Kodi 20, 21, 22.
- **Single test:** `pytest tests/test_items.py::test_watching`.
- **Build add-on zip:** `make video_addon VERSION=4.99.0` (VERSION is required; substituted into
  `addon.xml`). `make repo VERSION=...` also builds the Kodi-repository structure. Releases are
  driven by git tags via `.github/workflows/deploy.yaml`, which builds the repo and publishes it to
  Netlify with the official `netlify-cli` (`npx netlify-cli deploy`, needs `NETLIFY_*` secrets).

## Architecture

Kodi launches the add-on fresh on **every navigation action**, passing `sys.argv = [plugin_url,
handle, query_string]`. `src/addon.py` calls `plugin.run()`; there is no long-running process —
state that must survive between invocations is stashed in a Kodi window property (see Modeling).

**`Plugin` (`plugin.py`) is the central service-locator.** A single `plugin` instance (created in
`main.py`) parses `sys.argv` and wires together every subsystem: `settings`, `auth`, `logger`,
`routing`, `search_history`, `items` (the model factory), `client`, `proxy_settings`. Nearly every
other object holds a back-reference to `plugin` and reaches collaborators through it.

**Routing (`routing.py`) is a custom Flask-like router.** View functions in `main.py` are registered
with `@plugin.routing.route("/items/<content_type>/<heading>/")`. `Routing.dispatch` regex-matches
the incoming path and calls the matching view with captured kwargs. `build_url(...)` constructs
`plugin://` URLs that become directory item targets. Adding a screen = add a `@route`d function in
`main.py` that calls `xbmcplugin.addDirectoryItem(...)` / `endOfDirectory(...)`.

**API access (`client.py`).** `plugin.client("some/endpoint").get(data={...})` / `.post(...)` against
the kino.pub REST API. `KinoPubClient` builds a urllib opener with custom handlers that:
- inject the `Authorization: Bearer <access_token>` header,
- apply HTTP/SOCKS proxy settings read from **Kodi's system settings** via JSON-RPC
  (`xbmc_settings.py`),
- on **401** refresh the OAuth token and retry once, on **429** sleep and retry up to 3×.

**Auth (`auth.py`)** implements the OAuth2 **device-code flow** (show user a code + verification URL,
poll for token). Tokens are persisted through `Settings`.

**Modeling (`modeling.py`)** is the domain layer. `ItemsCollection` is a factory that turns API JSON
into model objects — `Movie`, `TVShow` (→ `Season` → `SeasonEpisode`), `Multi` (→ `Episode`) — chosen
via `CONTENT_TYPE_MAP` and the `subtype` field. Each model exposes `video_info`, `url`, and a
`list_item` (an `ExtendedListItem`). Key trick: because the add-on restarts each navigation, rendered
items are **pickled into the `10000` window property** `video.kino.pub-playback_data`
(`Plugin.set_window_property`). `instantiate_from_item_id` first tries that cache, only hitting the
API on a miss — this avoids refetching when the user clicks into an already-listed item.

**Playback (`player.py`).** `play` view sets a resolved URL and loops while a `Player(xbmc.Player)`
subclass receives playback callbacks. On stop/end it reports marktime / resume-point / watched status
back to the API, honoring Kodi's `advancedsettings.xml` thresholds (`ignoresecondsatstart`,
`playcountminimumpercent`, …) and sets Trakt scrobbling ids.

**Settings (`settings.py`).** `Settings.__getattr__`/`__setattr__` transparently proxy attribute
access to `xbmcaddon.Addon().getSetting/setSetting` (so `plugin.settings.access_token = "..."`
writes add-on settings). `show_*` attributes are eval'd to bool. `is_testing` (env `KINO_PUB_TEST`)
switches API base URLs between production and the local mock server.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quarckster/kodi.kino.pub](https://github.com/quarckster/kodi.kino.pub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
