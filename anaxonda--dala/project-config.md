---
trigger: always_on
description: - **`dala/` (Package)**:
---

# Repository Guidelines

## Project Structure & Module Organization
- **`dala/` (Package)**:
    - **`drivers/`**: Specialized extractors (HN, Reddit, Substack, YouTube, WordPress, Forum).
    - **`core/`**: Core logic including `ArticleExtractor` (text), `ImageProcessor` (media), and `DriverDispatcher`.
    - **`models.py`**: Shared data structures (`BookData`, `Source`, etc.).
- `main.py`: CLI entry point (replaces monolithic script).
- `web_to_epub.py`: Legacy shim for backward compatibility (imports from `dala`).
- `server.py`: FastAPI backend. Provides `/convert`, async `/jobs`, `/jobs/{id}/download`, `/helper/extract-links`, and `/helper/last-conversion`.
- `firefox_extension/`: Firefox add-on (Manifest V2).
- `extension_chrome/`: Chrome/Brave/Edge add-on (Manifest V3). Uses a shim and server-side parsing to bypass Service Worker DOM limits.
- `tests/`: Pytest coverage for dispatch, drivers, server conversion, server-side saving, and HN delegation behavior.
- `config/bpc/`: Ignored local checkout/extraction area for Bypass Paywalls Clean. Do not commit fetched third-party extension artifacts.

## Build, Test, and Development Commands
- `uv run main.py https://example.com/article`: Run the CLI directly.
- `uv run dala-server`: Start the FastAPI backend. Required for both Firefox and Chrome extensions. `uv run server.py` still works as a direct script fallback.
- `uv run pytest`: Run the automated test suite.
- `git status --short`: Check the dirty tree before editing and before handing work back.
- **Chrome Extension Dev:** Load `extension_chrome/` unpacked in `chrome://extensions`.
- **Packaging:** Run `./package_extensions.sh` to generate browser extension packages and the installer bundle in `dist/`.

## Chrome Extension (Manifest V3) Specifics
- **Service Worker:** Runs in `background.js`. No access to DOM/`DOMParser`.
- **HTML Parsing:** Offloaded to the local server (`/helper/extract-links`) via POST request.
- **Background Preparation:** `popup.js` sends `init_download`; `background.js` handles tab HTML capture, queue/bundle payload preparation, server job submission, polling, cancellation, and downloads. The popup can close after the background task starts.
- **Downloads:** Uses a fallback chain:
    1.  `URL.createObjectURL(blob)` (Standard, often blocked in SW).
    2.  `browser.downloads.download` with specific filename.
    3.  **Last Resort:** Converts Blob to **Data URI** (`data:application/epub...`) and downloads with `filename: fallback.epub`. This bypasses most filesystem/permissions issues.
- **Shim:** `chrome-shim.js` polyfills `browser.*` APIs to `chrome.*` APIs, wrapping callbacks in Promises.

## Known Limitations
- **Local Server State:** Async jobs are stored in process memory only. Server restart loses job status and temporary download paths.
- **Browser Fallback Policy:** Server browser fallback auto-detects Chrome, Edge, Brave, Chromium, or Playwright's managed Chromium, and uses the dedicated Dala profile at `~/.local/share/dala/browser-profile` by default. Do not auto-use the user's real browser profile. Interactive bot challenges should default to archive fallback; the extension can optionally open the original URL in the user's browser. The screenshot-based warm browser remains an explicit `browser_challenge_action: "warm"` path only. BPC-style extensions can be installed in the user's normal headed browser for extension capture; server-side BPC under `config/bpc/` is an advanced local fallback/testing path.
- **Server Trust Boundary:** CORS is open for extension use. Keep the server bound to `127.0.0.1` unless intentionally exposing it, and document remote/LAN server assumptions when changing extension server URL behavior.
- **Driver Heuristics:** Generic, forum, and image extraction include site-specific heuristics and fallbacks. When touching them, add focused regression tests with representative HTML.
- **Generated Artifacts:** EPUB/PDF outputs, extension packages, installer bundles, `web-ext-artifacts/`, `exports/`, pycache, screenshots, logs, local systemd exports, and `config/bpc/` are generated/local artifacts and should remain uncommitted unless explicitly requested for release staging.
- **PDF Image Assets:** EPUB assets may be optimized WebP, but PDF rendering should feed Chromium temporary JPEG assets derived from the selected image preset/color mode to avoid oversized embedded RGB image streams.

## Coding Style & Naming Conventions
- Python: 4-space indent, snake_case for functions/vars, CapWords for classes, prefer type hints and dataclasses where shared across API boundaries.
- Prefer async flows (aiohttp sessions) and explicit option objects over ad-hoc kwargs; keep network calls cancellable and timeouts explicit.
- Use f-strings for formatting; keep prints concise and debug-friendly.
- Respect EPUB/e-reader constraints: avoid modern CSS in generated content; keep layouts table-based when needed.

## Testing Guidelines
- Run automated tests before and after core/server/driver changes:
  - `uv run pytest`
- Add focused pytest coverage for driver routing, server payload handling, image URL normalization, and representative HTML fixtures when changing extraction behavior.
- Also use targeted manual runs for real-network and extension behavior:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anaxonda/dala](https://github.com/anaxonda/dala) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
