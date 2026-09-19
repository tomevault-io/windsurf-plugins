---
trigger: always_on
description: validates on assignment** — so `"false"` lands on a bool field intact and
---

# CLAUDE.md

Guidance for AI coding assistants (and humans) working in this repository.

# WarCounsel — Real-Time Log-Aware Assistant

A real-time companion for EverQuest Legends: tails the combat log, tracks the
character, and provides a live HUD (vitals, war ledger, encounters), an Atlas
(charts / mined geometry / textured 3D), and a wiki-grounded Advisor (spells,
AAs, gear, hunting spots). Passive by design — it never touches game files or
memory. User setup lives in **README.md**; this file is architecture,
invariants, and conventions.

**Stack**: FastAPI backend + polling log tailer + Next.js 14 frontend (app
router, TypeScript, hand-rolled CSS) + WebSocket live feed + optional LLM.

## Running for development

`start_companion.bat dev` — or by hand:

```
uvicorn backend.main:app --reload      # backend :8000 (from the repo root)
cd frontend && npm run dev             # UI :3000
```

END USERS run production mode (`start_companion.bat` with no args): uvicorn
WITHOUT --reload + `next start` serving the build from `.next-prod`
(~350MB lighter, no file watchers). Production builds use a SEPARATE dist
dir (`NEXT_DIST_DIR=.next-prod`), so a running dev server and a prod build
can no longer corrupt each other. The launcher **auto-rebuilds** the prod
UI when any frontend source is newer than the last build (a stale
`.next-prod` once served an old version). While iterating, prefer `dev`
mode — its `--reload` has occasionally wedged in production launches, and
a lite deterministic mode powers a planned single .exe (see below).

**Single executable** (`build_exe.bat` -> PyInstaller onefile, ~44MB,
~4s cold start; BUILT AND VERIFIED on Windows 11). Everything works
except screen OCR: HUD, overlay, Atlas 3D with textures, and LLM counsel.
FastAPI serves the static `frontend/out` at `/` (same-origin, `api.ts`
auto-detects; `next.config` `NEXT_EXPORT=1` static-exports — REBUILD IT or
the exe ships a stale UI); `run_companion.py` is the only entrypoint and
also dispatches the helper windows.

Hard-won, all of it load-bearing:

- **`backend/paths.py` owns the two roots.** `bundle_path()` = read-only
  assets from `sys._MEIPASS` (a temp dir wiped on exit); `data_dir()` =
  writable state beside the .exe, or `%LOCALAPPDATA%` when that is
  read-only. NEVER write state under the bundle. Source mode is unchanged
  (`./data`).
- **Helper windows go through `child_command()`**: frozen, `sys.executable`
  IS the app and `-m backend.overlay` would boot a second server, so the
  overlay/OCR calibrator use `--overlay` / `--ocr-overlay` flags.
- **Windowed builds have no console.** `sys.stdout` is None; uvicorn's
  colour formatter calls `.isatty()` on it, so `run_companion` adopts the
  streams onto `data/companion.log` and passes `log_config=None`. pywebview
  demands the MAIN thread, so uvicorn runs on a worker and closing the
  window shuts down through the normal lifespan.
- **Optional deps must never abort startup.** `ocr_system` catches
  `Exception`, not `ImportError` — a half-present rapidocr raises
  `FileNotFoundError`. Texture export failures degrade the 3D view to
  untextured rather than losing the zone.
- **`requirements-lite.txt` decides what the exe can do**, because
  PyInstaller only bundles what the BUILD MACHINE has installed. The LLM
  clients are in it deliberately: the settings panel offers an API key
  field. `llm_runtime.available()` probes at runtime and the panel greys
  out what is missing.
- **There is a SECOND packaged variant: `WarCounsel-OCR.zip`**
  (`requirements-heavy.txt`, `build_exe.bat heavy`, the `build-ocr` CI job
  — same app, screen OCR included). Three things about it are deliberate:
  - **It is `--onedir`, not `--onefile.`** A one-file bundle re-extracts
    its whole payload to a temp dir on EVERY launch — that IS the ~4s cold
    start — so the cost scales with size. At 200MB that tax would land on
    every start, which is also why OCR is not simply added to the lean
    build: the people who never enable it would pay for it forever.
  - **The size is opencv, not the OCR engine.** Measured 2026-08-08:
    cv2 112.4MB, onnxruntime 42.5, numpy 30.8, rapidocr 15.6, mss 0.4 —
    204.6MB total, against a 43.6MB exe. `mss` is 0.4MB and is never the
    problem, whatever the error message suggests. **opencv-python-headless
    does NOT help** (112.0MB — the same 82MB `cv2.pyd` on Windows; the
    ~40MB figure quoted for it is the WHEEL). What IS droppable is
    opencv's 29.4MB `opencv_videoio_ffmpeg*.dll`, a video codec that OCR
    on still screenshots cannot reach; the build deletes it.
  - **CI asserts `/api/ocr/status.deps_ok` on the built artifact.** The
    import guard in `ocr_system` is broad on purpose (a half-present
    rapidocr raises `FileNotFoundError`, not `ImportError`), so a broken
    OCR bundle looks exactly like a working one until someone enables it.
    `deps_ok` alone proved too weak — `run_companion.py --ocr-check`
    (mirroring `--overlay-check`) runs the REAL engine over a rendered
    "X: 1234" and fails unless the digits come back.
  - **Build the OCR release on PYTHON 3.12.** The engine renamed itself and
    the two packages are not interchangeable: 3.12 gets
    `rapidocr-onnxruntime` v1, which SHIPS its ONNX models; 3.13 gets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EKirschmann/WarCounsel](https://github.com/EKirschmann/WarCounsel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
