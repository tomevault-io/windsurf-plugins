---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Pipeline that packs global forecasts (2 m temperature `tmp2m`, precipitation rate `prate`, 10 m wind `ugrd10m`/`vgrd10m`, solar radiation `dswrf` on sflux only, f000–f240 on a mixed-step axis) into a custom per-variable spatiotemporal binary format (Xue, `.xue`), plus a static MapLibre page that decodes it with a Rust WASM worker and renders via a custom WebGL2 layer (inverse Web Mercator + palette lookup done in the shader). Three sources share the whole pipeline (`--model gfs|ecmwf|sflux`, registry in `sources.py`): NOAA GFS 0.25° (hourly to f120, 3-hourly to f240 — 161 frames), ECMWF IFS open data (3-hourly to 144 h, 6-hourly to f240 — 65 frames; CCSDS-packed GRIB repacked via eccodes `grib_set` at fetch time because GDAL builds often lack libaec, and prate derived by de-accumulating `tp`), and GFS sflux on the native ~13 km Gaussian grid (same cadence as GFS; planes rolled to the −180-first layout at extract time; prate de-averaged from window-cumulative `PRATE ave` records, so its axis starts at f001; adds the `dswrf` bundle). Each model is an independent dataset: `gfs.<run>/` + `latest.json`, `ecmwf.<run>/` + `latest-ecmwf.json`, `sflux.<run>/` + `latest-sflux.json`. The wind pair ships as one two-variable `wind10m.xue` bundle (optional in inputs and manifest, like `dswrf`) and renders through a GPU particle layer. Three languages, one format:

- **Python encoder** (`xue/`): fetch → convert → quantize → temporal residuals → zstd → container. GDAL and ffmpeg are invoked as CLI subprocesses (`gdal.py`, `ffmpegcli.py`) — no binary Python dependencies. zstd (`zstdcli.py`) uses the stdlib `compression.zstd` in-process on Python ≥ 3.14 (per-plane subprocess overhead was the build bottleneck) with a zstd-CLI fallback for older interpreters; bundles are written and read-back-verified concurrently through one shared machine-sized pool. Band discovery reads GRIB2 section 0/1/4 headers directly (`grib2.py`, ~1 ms/file vs ~1 s/file for `gdalinfo -json`); one real gdalinfo pass per run doubles as the wind-availability probe and a cross-check of the header index, and any parse failure or disagreement falls the run back to full gdalinfo inspection.
- **Rust decoder** (`rust/xue` core crate; `rust/xue-wasm` bindings). `make wasm` builds it into `web/src/wasm/` (gitignored generated output consumed by the frontend).
- **TypeScript frontend** (`web/src/`): `main.ts` orchestrates; `manifest.ts` resolves the mutable `latest.json` live pointer → per-run `manifest.json` and picks a resolution tier (`pickBundleVariant`); `worker.ts` owns WASM decode, a byte-budgeted plane cache, and windowed prefetch; `layer.ts` is the MapLibre custom layer with two-texture blend playback; `particles.ts` is the wind GPU particle layer (u/v interleaved into one RG8 texture, ping-pong state/trail textures, speed palette); `poster.ts` decodes fast-first-frame posters via `DecompressionStream`; `webcodecs.ts` is the alternate H.264 path (any variable; used only when the stream is not larger than the `.xue`); `i18n.ts` is the zh/en UI dictionary (locale = `?lang=` param > persisted footer-toggle choice > `navigator.language`, fixed per page load; the Protomaps basemap label language follows it; thrown Error diagnostics stay English in both locales).

The format spec is `docs/format.md` (normative — update it if the format changes). The README is `README.md` (English).

## Commands

`make check` verifies external tools: Python ≥3.12 (use `uv sync` → `.venv`), GDAL ≥3.8 with GRIB driver, zstd CLI, Node ≥22, Rust + `wasm-pack`.

Build and run:

```sh
make mvp                 # fetch latest run, build .xue bundles + manifest, build frontend
make mvp RUN=2026081600 HOURS=120 PROFILE=balanced FORCE=--force
make mvp MODEL=ecmwf     # same, from ECMWF IFS open data (needs eccodes grib_set)
make serve               # vite preview of dist/
npm run dev              # dev server (reads web/public/data/ directly)
make wasm                # rebuild WASM decoder after Rust changes
```

CLI equivalents: `python -m xue {fetch,convert-bin,verify-bin,build-bin}` (see README for arguments). Re-writing an existing manifest requires `--force`.

Tests:

```sh
make test                # Rust + Python + vitest (runs prepare_bin_fixture.py for you)
make test-e2e            # Playwright (once: npx playwright install chromium)
python -m unittest tests.test_bin -v                       # one Python module
python -m unittest tests.test_bin.ClassName.test_name -v   # one test
npx vitest run tests/web/manifest.test.ts                  # one web unit test
npx playwright test -g "pattern"                           # one e2e test
cd rust && cargo test                                      # Rust only — run tests/prepare_bin_fixture.py first
wasm-pack test --headless --chrome rust/xue-wasm        # decoder in a real browser
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ringsaturn/xue](https://github.com/ringsaturn/xue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
