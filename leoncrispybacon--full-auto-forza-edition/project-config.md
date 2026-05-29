---
trigger: always_on
description: A Windows desktop automation tool for Forza Horizon 6. Built with Python + CustomTkinter. Uses a hybrid OpenCV + optional OCR detection pipeline to identify game screens and simulate keyboard/mouse input.
---

# Full Auto Forza Edition (FAFE) — Project Summary

## What is this?
A Windows desktop automation tool for Forza Horizon 6. Built with Python + CustomTkinter. Uses a hybrid OpenCV + optional OCR detection pipeline to identify game screens and simulate keyboard/mouse input.

## File Structure
```
New APP/
├── forza_app.py          # Entry point
├── main_window.py        # Main UI window (CTk), all tabs, settings panel
├── setup_panel.py        # Collapsible Setup & Templates panel with threshold sliders
├── capture.py            # Screenshot capture, region selection, CaptureSession, NodeSession
├── detector.py           # ScreenDetector — ROI + multi-scale + edge + optional OCR matching
├── race.py               # Race Auto-Grind automation logic
├── mastery.py            # Auto Unlock 22B Mastery automation logic
├── delete_cars.py        # Delete Used Cars automation logic
├── log_widget.py         # Thread-safe log widget with colored warning support
├── app_lang.py           # All UI strings in zh-tw / zh-cn / en
├── config.py             # Config load/save, path helpers
├── updater.py            # GitHub release auto-updater
├── version.py            # VERSION = "1.2.1"
├── build_app.bat         # PyInstaller build script
├── templates/            # Template images (race/, mastery_full/, each with 1080p/1440p/2160p/custom)
└── index.html            # GitHub Pages intro/tutorial page (trilingual)
```

## Architecture

### UI (main_window.py)
- 4 tabs: Race Auto / Auto Unlock 22B / Auto Buy 22B-STi / Delete Used Cars
- Settings is inline (not popup) — ⚙ hides topbar+tabs and shows settings panel, ← Back restores
- Topbar: app title, monitor picker, ⚙ button, ☕ Support Me button (bottom-right overlay)
- Each tab has: description label, optional count input (0=unlimited), Start/Stop buttons, log widget
- Auto Unlock 22B tab also has a start-row selector (CTkSegmentedButton, values 1/2/3) between count and run controls; saved as `mastery_start_loop` in config
- Language: zh-tw (default), zh-cn, en — switching triggers app restart
- Theme: system/light/dark

### Detection (detector.py)
- `ScreenDetector.detect(frame, key, template, threshold)` returns a `MatchResult` dataclass
- ROI-first: each template key has a configured search region (see `DEFAULT_ROIS`); full-screen fallback applies a 0.94 score penalty
- **Text template fast path**: `TEXT_TEMPLATES = frozenset(DEFAULT_ROIS.keys())` — all 11 keys. Text templates skip the Canny edge channel and use only 3 scales (`detector_text_scales`, default `[0.95, 1.00, 1.05]`), reducing matchTemplate calls from 14 to 3 (~4–5× faster). Non-text templates keep the full 7-scale + edge pipeline.
- Full pipeline scales: `[0.86, 0.92, 0.97, 1.00, 1.03, 1.08, 1.14]`
- Hybrid score (non-text): `0.62 × grayscale(equalizeHist) + 0.28 × Canny edges + up to 0.10 OCR bonus`
- Text score: `grayscale(equalizeHist)` only (edge channel skipped)
- **OCR-primary mode** (beta, default off — the real cross-hardware fix): For text templates with OCR hints, OCR becomes a first-class confirmation signal rather than a +0.10 bonus. Pixel template matching of game UI text is fragile across hardware (GPU AA, font hinting, HDR, game settings all change pixels) — text content is what's actually invariant. Logic: if `image_score >= detector_ocr_skip_above` (default 0.75), skip OCR for the fast path; else run OCR; if hint matched, promote score to `max(image_score, detector_ocr_confirm_score)` (default 0.85). To prevent the per-key OCR cooldown from breaking the stability filter, an OCR confirmation is **cached** per key for the cooldown duration and applied to subsequent frames without re-running OCR (gated by `detector_ocr_cache_pixel_min`, default 0.15, to guard against the screen changing during cooldown). Enable via `detector_ocr_primary: true`. Legacy default behaviour: OCR only in narrow borderline window as +0.10 bonus.
- **OCR pre-warming**: `OptionalOCR._ensure_loaded()` is kicked off in a background thread when `ScreenDetector` is constructed, so the first detection call doesn't eat the 1–2s onnxruntime model-load cost.
- **OCR cooldown**: `detector_ocr_cooldown` (default 1.0s) — minimum gap between actual OCR calls per template key. Prevents CPU spikes during long failed-detection streaks. Under OCR-primary, the confirmation cache uses this same duration.
- **Warning log surfaces OCR text**: when detection stalls for 3s, the warning includes `OCR: 'xxx'` showing what OCR read in the borderline zone — critical for diagnosing why a template isn't matching on a given user's machine.
- Soft threshold: actual cutoff is `max(0.45, threshold * 0.92)` — user's slider is a target, not a hard wall
- Stability filter: requires N consecutive frames (default 2) above the soft threshold; `stable=False` flag bypasses this for single-shot click ops
- Optional OCR: lazy-loads `rapidocr_onnxruntime` → `rapidocr` → `pytesseract`. Hint words in `OCR_HINTS` are multilingual (en/zh-tw/zh-cn). If no OCR backend is installed, detection still works

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leoncrispybacon/Full-Auto-Forza-Edition](https://github.com/Leoncrispybacon/Full-Auto-Forza-Edition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
