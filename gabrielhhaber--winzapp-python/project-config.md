---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

WinZapp is a free, self-hosted Windows desktop WhatsApp client built specifically for **accessibility** (blind/low-vision users via NVDA/JAWS/Narrator through `accessible_output2`). It's a hybrid app: a Python 3.13 + wxPython GUI process drives a locally-run **WPPConnect Server** (Node.js, cloned/built from the upstream `wppconnect-team/wppconnect-server` repo) that acts as the actual WhatsApp Web gateway. The two processes talk over local HTTP REST (`http://127.0.0.1:6300/api/...`) and Socket.IO (real-time events).

## Commands

### Dev setup
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install -r requirements-dev.txt   # adds pytest, pytest-cov, pytest-asyncio
python setup_api.py                   # clones + builds client/api/ (WPPConnect Server) — one-time, requires Node
```
`setup_api.py` clones WPPConnect Server into `client/api/`, restores WinZapp's custom patched files (`start.js`, `config.json`, plus `src/config.ts`, `src/index.ts`, `src/util/{createSessionUtil,sessionUtil,functions}.ts`, `src/middleware/statusConnection.ts`, `src/controller/{deviceController,messageController,sessionController,statusController}.ts`, `src/routes/index.ts`, `decrypt.js` — the full, current list is `CUSTOM_ROOT_FILES + CUSTOM_SRC_FILES` at the top of the script), then runs `npm install` and `npm run build` inside `client/api/`. `package.json` is handled separately by `_merge_package_json_dependencies()` (overrides only WinZapp's specific dependency entries, not a full-file restore — see that function's own docstring for why). Re-run it any time `client/api/` needs to be rebuilt — it preserves `node_modules` and the custom files across re-clones. `build.py` also auto-detects when `client/api/` has drifted from `client/api_patches/` (a patch edited but this script never re-run) and re-runs it automatically before compiling — see the Packaging section below.

### Run the client in dev mode
```powershell
cd client
python main.py
```
Entry point is `client/main.py`, guarded by `if __name__ == "__main__":` near the bottom of the file. There is no separate "start the API server" dev command — `main.py` launches/manages the local Node WPPConnect Server process itself.

### Tests
```powershell
pytest                                   # from repo root; pytest.ini sets pythonpath=client, asyncio_mode=auto
pytest tests/test_database.py            # single file
pytest tests/test_database.py::TestChats::test_upsert_chat_creates_record  # single test
```
Tests cover `client/core/database.py` and `client/core/database_bridge.py` (async SQLite layer + its sync façade) and small islands of pure logic pulled out of `main.py`/`core/notification_manager.py`/`ui/conversations.py` (e.g. `tests/test_sender_names.py`, `tests/test_notifications.py`, `tests/test_delivery_status.py`, `tests/test_send_jid_resolution.py`, `tests/test_message_bookmarks.py`) by binding their unbound methods onto a plain stub object — `MainWindow`/`ConversationsPanel` are wx.Frame/wx.Panel and can't be instantiated without a running wx.App, so the stub carries only the attributes the method under test actually touches. There are no tests for the wx UI in bulk. Async tests use `pytest-asyncio` in `auto` mode — no `@pytest.mark.asyncio` decorator needed, just declare test functions `async def`. **New functions/features should come with a test in this style as part of the same change.**

### Building the distributable
```powershell
venv\Scripts\python.exe build.py             # onedir: WinZappInstaller.exe + WinZapp.zip
venv\Scripts\python.exe build.py --onefile   # single-file WinZapp.exe + WinZapp.zip
```
Requires, in addition to the venv: `client/node/` (portable Windows x64 Node.js extracted there), `client/api/dist/server.js` built (via `setup_api.py`), and — for `--onedir` only — `gcc`/`windres` in `PATH` (MSYS2 UCRT64) to compile the C installer/uninstaller stubs in `installer/`. `client/api/` and `client/node/` are git-ignored and must be prepared locally before building; see `.github/workflows/release.yml` for the exact CI sequence if reproducing a release build. `check_tools()` (step 1) also diffs every patched file in `client/api_patches/` against its live copy in `client/api/` and, on drift, re-runs `setup_api.py` automatically before continuing — a patch edited only in `client/api_patches/` without rebuilding used to ship a stale/reverted `dist/server.js` with no warning.

## Architecture

### Two-process split
- **`client/` (Python/wxPython)** — all UI, business logic, local persistence, notifications, sounds. This is what you'll be editing almost all of the time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielhhaber/WinZapp_Python](https://github.com/gabrielhhaber/WinZapp_Python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
