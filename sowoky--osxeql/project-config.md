---
trigger: always_on
description: **What this is:** a macOS app that runs **EverQuest Legends** on Apple Silicon using
---

# osxEQL — project rules (read this first)

**What this is:** a macOS app that runs **EverQuest Legends** on Apple Silicon using
**open-source Wine + DXMT** (DirectX 11 → Metal). Goal: fully open-source and shareable,
**independent of CrossOver**. We reverse-engineered CrossOver to learn the mechanism, then
rebuilt it from OSS parts.

**STATUS: shipped and proven cold.** The v0.3.0 DMG carried a wiped, never-ran Mac from
download → installer → login → 6 GB game download → in game, in ONE launch (2026-07-12,
Kyle: "worked perfect"). Wine runtime is our own compile of CodeWeavers' published LGPL
source (upstream-vanilla was a dead end — DXMT's macdrv ABI needs the CrossOver lineage);
the DMG is self-contained (bundled dylib closure + MoltenVK ICD + native setup window).
See [`docs/STATUS.md`](docs/STATUS.md) and [`docs/LAUNCHPAD-LOGS.md`](docs/LAUNCHPAD-LOGS.md).

## One-paragraph mental model
EQL's `eqgame.exe` is **64-bit** and renders with **DirectX 11**. To run it on macOS you
need (1) **Wine** to run the Windows binary and (2) a **D3D11→Metal translator**. CrossOver
uses Apple's proprietary **D3DMetal** (license forbids shipping it). We use **DXMT**
(github.com/3Shain/dxmt, open source) instead. DXMT needs exactly ONE special thing from
Wine: the **`macdrv_functions`** symbol exported from `winemac.so` — its bridge to attach a
Metal view to the Wine window. Stock Wine doesn't export it; CrossOver's build and 3Shain's
patched Wine do. **That symbol is the crux of this whole project.**

## Where everything lives
- **The app:** `/Applications/osxEQL.app` — double-click → Daybreak LaunchPad → log in →
  Play → game renders via DXMT. Since v0.2.1 the bundle also carries the Homebrew dylibs
  wine dlopens (`Wine/lib/lib*.dylib`, staged by `packaging/bundle-dylibs.sh`), so the DMG
  runs on Macs with no Intel Homebrew.
- **The runtime ("bottle"):** `~/Library/Application Support/osxEQL/`
  - Since the 2026-07-12 clean-Mac wipe + fresh-DMG test, kyle-mac looks like a USER
    machine: **no staged `Wine/` dev runtime, no `prefix-cx/`** — the wine runtime lives
    ONLY inside `/Applications/osxEQL.app/Contents/Resources/Wine` (build-app.sh falls
    back to it as WINE_SRC), and Intel Homebrew is gone (reinstall it + rerun
    `engine/build-wine.sh` if a runtime rebuild is ever needed).
  - The runtime is our **self-built CrossOver 26.2.0 Wine** — compiled from CodeWeavers'
    official published LGPL source with system clang (`engine/build-wine.sh`). Has the
    `macdrv_functions` bridge natively; no CrossOver install needed. Drive it via
    `Wine/bin/wine` (the real loader) — **do NOT set `WINELOADER`** (gotcha #2).
  - `prefix/` — **the ACTIVE prefix**: holds the EQ client (installed via the app's own
    first-run flow). The launcher prefers it whenever `prefix/system.reg` exists;
    `prefix-cx` is a legacy fallback name only.
- **The engine:** `engine/` — the `osxeql` CLI + numbered scripts. Works headless; the app
  is a thin shell over it.
- **Docs:** [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) (deep technical),
  [`docs/STATUS.md`](docs/STATUS.md), [`docs/JOURNEY.md`](docs/JOURNEY.md) (why each
  decision), [`docs/VISION.md`](docs/VISION.md).

## Hard-won gotchas — DO NOT re-derive these
1. **`macdrv_functions` is everything.** DXMT's `winemetal.so` looks it up from `winemac.so`
   to create the Metal view (`macdrv_view_create_metal_view` / `get_metal_layer` /
   `release_metal_view`). No symbol → `err: Failed to create metal view`. Verify:
   `nm -gU "$WINE/lib/wine/x86_64-unix/winemac.so" | grep macdrv_functions`. Only
   CrossOver-build / 3Shain-patched Wine has it; stock Gcenx does NOT. The full prebuilt
   matrix (every dead end) is in `docs/JOURNEY.md` — don't re-hunt prebuilts.
2. **Drive the self-built Wine via `bin/wine`, and do NOT set `WINELOADER`.** In our
   from-source build `bin/wine` IS the real Mach-O loader (the *extracted-from-CrossOver*
   build was different: its `bin/wine` was a Perl wrapper needing a "bottle", so that one
   used `bin/wineloader`). With a from-source `bin/wine`, **setting `WINELOADER` makes wine
   copy the loader to a temp dir for child processes** (explorer→LaunchPad→eqgame) which
   then fail `could not load ntdll.so`. Set `WINEPREFIX / WINESERVER / WINEDLLPATH` and run
   `bin/wine` directly; leave `WINELOADER` unset. See `engine/04-launch.sh` + the app script.
3. **DXMT install has THREE placements.** Builtin DLLs (`d3d11, d3d10core, dxgi, winemetal`)
   → `<wine>/lib/wine/{x86_64,i386}-windows/`; `winemetal.so` → `.../x86_64-unix/`; AND
   `winemetal.dll` ALSO copied to `<prefix>/drive_c/windows/system32/`. Missing the last one
   = `Unable to load EQGraphics.DLL (126)` (dependency-not-found cascade).
4. **eqclient.ini is CRLF; the invariant is `ini sizes == the Wine virtual desktop size`.**
   Anchored `sed`/regex without `\r` silently no-ops — use Python with `\b`. Any mismatch →
   mouse-cursor offset (EQ maps clicks in ITS resolution inside a differently-sized surface)
   AND the fullscreen-exclusive popup. EQ has TWO size key pairs: `WindowedWidth/Height`
   (windowed) and `Width/Height` (in-game fullscreen) — the `.app` pins ALL FOUR plus the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sowoky/osxEQL](https://github.com/sowoky/osxEQL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
