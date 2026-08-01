---
trigger: always_on
description: QiTV — Agent Guide and Working Plan
---

QiTV — Agent Guide and Working Plan

Overview
- Purpose: Maintain a clear, shared plan and conventions for ongoing refactors and fixes.
- Scope: Applies to the entire repository unless a more specific AGENTS.md is present in a subdirectory.

Principles
- Keep the UI responsive: no blocking network or heavy I/O on the UI thread.
- Fix root causes, not symptoms; keep changes small and focused.
- Prefer composition over monoliths; split large modules by responsibility.
- Use consistent logging over prints and propagate errors meaningfully to the UI when needed.
- Write code that’s testable; isolate logic from PySide UI where possible.

Style & Tooling
- Format: black + isort (configured via pre-commit).
- Lint: flake8 (treats most issues as warnings except syntax/undefined names).
- Types: add gradual type hints; keep mypy green on changed modules.
- Logging: use `logging.getLogger(__name__)` rather than `print`.

Current Work Plan (Living TODO)
1) Input/UI polish and correctness
   - [x] Separate dblclick fullscreen from single-click play/pause (video_player.py)
   - [x] Remove unused `installEventFilter(self)` on `video_frame` or implement `eventFilter` explicitly
   - [x] Normalize progress bar behavior for live/VOD; avoid toggling visibility repeatedly
   - [x] Add keyboard shortcuts as QActions (Play/Pause, Mute, Fullscreen, PiP) and bind menu/toolbar if added later

2) Networking and responsiveness
   - [x] Identify and thread key `requests` (M3U load, STB categories, link creation)
   - [x] Standardize timeouts/retries across network calls (added timeouts; moved update check to QThread)
   - [x] Move remaining UI-thread `requests` to workers (exports OK as is)
   - [x] Ensure all worker completions marshal back to the UI thread (no cross-thread timers)
   - [x] Consolidate provider/EPG URL building and headers in one place

3) Modularity and structure
   - [x] Extract delegates to `widgets/delegates.py`
   - [x] Move M3U parsing to `services/m3u.py`
   - [x] Move export helpers to `services/export.py`
   - [ ] Split remaining `channel_list.py` into widgets/ (panels) and services/ (provider, epg)
   - [ ] Move EPG parsing unit-testable logic out of UI code paths
   - [x] Refactor image pipeline: workers only cache bytes; GUI builds QPixmap/QIcon on main thread
   - [ ] Consider a small event-bus/signal helper to decouple UI components
 - [x] Add network security toggles (Prefer HTTPS, SSL verify) and plumb into requests/aiohttp (XTREAM/STB/M3U)
 - [ ] Debug log EPG endpoints for providers (STB `load.php` get_epg_info, Xtream `xmltv.php`) to help diagnose ID mismatches
 - [x] Show all EPG entries in channel program list (no windowing), with local time formatting
 - [x] Investigate VideoPlayer seeking: prevent playback ending on seek; ensure double-clicking the progress slider controls the bar (not window drag)
 - [ ] Investigate optional VLC quality enhancements for low-bitrate streams (audio compressor/equalizer and lightweight upscaling/sharpening), gated by settings and disabled by default

4) Logging and error handling
   - [x] Add module-level loggers; remove stray prints
   - [x] Downgrade transient image fetch errors to info; reduce log noise
   - [ ] Plumb important errors to the UI via signals (non-modal first, modal where necessary)

5) Testing and stability
   - [ ] Add tests for provider cache pruning and image cache accounting
   - [ ] Add tests for XMLTV parsing and MultiKeyDict behavior
   - [ ] Add simple smoke tests for content loader pagination/aggregation

6) Packaging & config
   - [x] Completing the Github Actions for UV environment usage.
   - [ ] Pin more dependency versions in requirements.txt (PySide6, orjson, aiohttp, tzlocal)
   - [x] Add a `pyproject.toml` for tool config (black/isort/mypy) to keep settings centralized
   - [x] Drive bundle/app version from `pyproject.toml` in PyInstaller specs

Next Steps (Paused)
- Extract panels from `channel_list.py` into `widgets/`:
  - content info panel, list panel, media controls
- Add `services/provider_api.py` to centralize STB/Xtream calls with timeouts + QThread wrappers
- Move remaining UI-thread `requests` to workers (exports may stay synchronous)
- Introduce lightweight dataclasses for Channel/Program for safer data access
- Add cancelation support to network workers (or switch to aiohttp within QThreads)
- Add unit tests for `services/m3u.py` and `services/export.py`

Recent Changes (for context)
- UX: Added QActions for playback controls (Space: Play/Pause, M: Mute, F: Fullscreen, Alt+P: PiP) for future menu/toolbar binding (video_player.py)
- UX: Normalized VOD vs Live progress behavior; avoid repeated visibility toggles and only update values on VOD (video_player.py)
- Refactor: Centralized STB URL building in `services/provider_api.py`; updated STB workers and EPG to use it (channel_list.py, epg_manager.py)
- Fix: Eliminated cross-thread timer warnings by posting worker completions to the GUI thread (channel_list.py: M3U/STB/link creators; update_checker.py). Also avoided unconditional signal disconnects that caused warnings.
- Refactor: Image loading pipeline avoids GUI objects in worker threads; workers cache files, GUI constructs QPixmap/QIcon (image_loader.py, image_manager.py, channel_list.py logos/posters).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ozankaraali/QiTV](https://github.com/ozankaraali/QiTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
