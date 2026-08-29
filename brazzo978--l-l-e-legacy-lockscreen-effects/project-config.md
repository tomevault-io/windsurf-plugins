---
trigger: always_on
description: - Git root: `D:\New project`.
---

# Agent Notes

## Repository and branches
- Git root: `D:\New project`.
- GitHub: `https://github.com/Brazzo978/unlock-effects-test` private repo.
- Active work branch: `codex/lle-unified`.
- Stable validated tag: `charging-lock-stable-perfect-2026-06-30`.
- Touch baseline tag: `charging-touch-advanced-baseline-2026-06-30`.

## Unified LLE trunk (2026-07-15)
- Canonical application path: `LLEUnified`.
- Package: `com.codex.lle`.
- All new Java logic, resources, preferences and UI changes must be made in
  `LLEUnified`; do not implement them separately in the frozen ARM32/ARM64 trees.
- Frozen pre-unification tag: `lle-pre-unification-2026-07-15`.
- Frozen reference trees:
  - ARM32: `unlock-effects-test\charging-touch-test-apk`.
  - ARM64: `LLE64`.
- Unified build commands:
  - both: `powershell -ExecutionPolicy Bypass -File .\LLEUnified\build.ps1`;
  - ARM32: add `-Target Arm32`;
  - ARM64: add `-Target Arm64`.
- Outputs:
  - `LLEUnified\build\armeabi-v7a\LLE-armeabi-v7a-debug.apk`;
  - `LLEUnified\build\arm64-v8a\LLE-arm64-debug.apk`.
- Runtime availability must use `EffectAvailability` and actual process bitness.
  Never load an ARM32 library from an ARM64 process or vice versa.

## Legacy apps
- Old app modules must not be revived or targeted by PRs.
- Legacy charging-only app modules were removed from the active repo.
- Port useful code into LLE manually instead of merging/cherry-picking legacy app changes.

## Frozen pre-unification ARM32 LLE reference
- Path: `unlock-effects-test\charging-touch-test-apk`.
- Package: `com.codex.lle`.
- Current APK: `unlock-effects-test\charging-touch-test-apk\build\LLE-debug.apk`.
- LLE means Legacy Lockscreen Effect; it is the experimental branch for touch listening and unlock FX.
- This path is historical after 2026-07-15; port useful findings into
  `LLEUnified` instead of developing here.

## LSE app
- Path: `unlock-effects-test\demo-apk`.
- Package: `com.codex.s4unlockfx`.
- Launcher label: `L.S.E`.
- LSE means Legacy Samsung Effect; keep this demo/reference app alongside LLE.
- Current features:
  - Touch box calibration from app UI via `TouchBoxSetupActivity`.
  - Transparent calibrated touch window using `TouchDebugView`.
  - Optional `Charging doodle overlay` toggle to hide doodles during FX testing.
  - S4 raw sounds copied into `res/raw`: `lens_flare_tap.ogg`, `lens_flare_unlock.ogg`.
  - Current active lens flare path is the original Samsung S4 visual effect dex loaded by `LensFlareEffectView`.
  - Current gesture flow: effect starts on `ACTION_DOWN`, follows `MOVE`, opens PIN only after swipe distance threshold.
  - PIN opening is attempted with Accessibility `dispatchGesture`; service XML includes `android:canPerformGestures="true"`.
- Important separation: charging doodles and unlock FX are separate systems.
  Doodles remain gated by real charging state; unlock/touch FX must work on the lockscreen even when not charging.
- 2026-07-02 active-effect correction:
  - Current validated/reliable effects are S4 Lens Flare Canvas port and S5 Popping Colours.
  - Picker order requested by user: `S3 ripple WIP`, `S4 Lens Flare`, `S5 Popping Colours`, `N4 Watercolor WIP`.
  - Watercolor picker value `3` is an active WIP slot routed to `WatercolorEffectView`, an app-owned transparent renderer. It uses the original Watercolor mask/tube/noise assets and lockscreen screenshot only as a color map, but it is still not exact and must not be presented as faithful Samsung parity.
  - Watercolor must stay visually transparent outside the local brush marks; do not route it through a native/Surface full-screen renderer that can blacken or repaint the lockscreen.
  - S5 coloured droplets and S5 sparkling bubbles were removed from the active app after phone testing confirmed they are broken/blacken the lockscreen.
  - `SamsungNativeEffectView.java` was removed from the active source. Do not route Watercolor/droplet/bubbles back through the direct Samsung native wrapper.
- 2026-07-03 sync note:
  - GitHub had commits `e1437d0` / `175b9d2` adding an S4 screen-on center hint, but they were based on an older touch app state.
  - Do not merge those commits blindly over the local WIP. The useful S4-only hint logic was manually re-applied on top of the latest local app state.
  - The screen-on hint is a generic renderer hook: 500 ms after `ACTION_SCREEN_ON`, schedule `showUnlockAffordance` at the center of the effect overlay.
  - S4 Lens Flare handles it with the app-owned Canvas tap burst/fog. S5 Popping Colours handles it through Samsung `handleCustomEvent(1, {"StartDelay","Rect"})` after its background color map is ready. N4 Watercolor WIP currently keeps a no-op hint until the effect itself is faithful enough.
- 2026-07-03 S3 Ripple reverse update:
  - Treat S3 ripple as a separate app-owned Fluid/ripple renderer, not a Samsung native wrapper.
  - Original uses OpenGL mesh plus height/velocity arrays and native `Fluid` functions (`Ripple_Render`, `Update`, `Advect*`, `Jacobi`, `AddInk`, `AddVelocity`) to produce refraction/reflection/specular.
  - Saved reverse reports:
    - Java/smali-side parameters: `s3ripplereverse\s3_ripple_smali_params_2026-07-03.md`.
    - Native/GL extraction from agent 2: `s3ripplereverse\s3_ripple_native_extraction_agent2_2026-07-03.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Brazzo978/L.L.E-Legacy-Lockscreen-Effects](https://github.com/Brazzo978/L.L.E-Legacy-Lockscreen-Effects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
