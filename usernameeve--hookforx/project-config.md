---
trigger: always_on
description: LSPosed/Xposed (Kotlin, API 82) module that removes ads from X (Twitter) (`com.twitter.android`). Single module `:app`. AGP 8.1.0 / Kotlin 1.9.0 / Gradle 8.2 / JDK 17, minSdk 27, target 34.
---

# AGENTS.md

LSPosed/Xposed (Kotlin, API 82) module that removes ads from X (Twitter) (`com.twitter.android`). Single module `:app`. AGP 8.1.0 / Kotlin 1.9.0 / Gradle 8.2 / JDK 17, minSdk 27, target 34.

## Build / deploy / verify

```bash
./gradlew assembleRelease                          # outputs UNSIGNED apk
apksigner sign --ks ~/.android/debug.keystore --ks-pass pass:android \
  --out app/build/outputs/apk/release/app-release.apk \
  app/build/outputs/apk/release/app-release-unsigned.apk
adb install app/build/outputs/apk/release/app-release.apk
adb logcat -s HookForX
```

- **After every `adb install -r`, re-enable the module in LSPosed Manager** (scope: `com.twitter.android`).
- Expected startup log: `HookForX loaded into process: com.twitter.android` + `HookResolver: <target> resolved via cache|known|scan → <class>` + `RepositoryFilter: k.b/emit ad-block installed` + `l.v presenter blocker installed` + `PremiumProbe: installed (override=..., flag getters=N, subscription checks=N)` + `HookRegistry: 4 hookers processed`.
- Tool paths: adb = `C:\Users\Administrator\Android\platform-tools\adb.exe`, jadx = `C:\Users\Administrator\jadx\bin\jadx.bat`. JDK 17 required (`C:\Program Files\OpenJDK17`). SDK located via `ANDROID_HOME`.
- No tests, no lint/CI — verification is manual on-device via logcat + uiautomator.
- **Logcat buffer floods fast** (X logs + emit counters): enlarge with `adb logcat -G 32M` before long verification sessions; keep emit/k.b logs throttled (every 20th).

## Architecture

`MainHook` (entry, declared in `assets/xposed_init`) → `DexKitLoader.load()` (`System.loadLibrary("dexkit")` first — LSPosed mounts the module's native libs onto X's linker path; falls back to extracting `libdexkit.so` from the module APK into X's code_cache + `System.load`; failure degrades to known-targets-only) → `HookRegistry.install()` → each `BaseHooker` independently in its own try/catch (one failure never blocks the others):
- **`resolver/HookResolver.kt`** — version-resilience layer resolving every obfuscated hook target structurally. Chain per target: `hookforx_force_scan` ON → skip to scan; else disk cache `files/hookforx/obfs_v<TABLE_VERSION>.json` (composite-CRC-keyed, sub-ms) → KnownTargets table (`NameMapping.kt` `KNOWN`, composite CRC matches a verified version) → DexKit fingerprint scan / chained structural derivation → structural validator (4 targets) → on failure `XLog.e` (target + fingerprint + reason) + null (soft fail, other hookers unaffected). Chained targets are structure-derived, never name-matched: `presenter_a` ← `lv.v(int, UrtTimelineItem)` return type; `urt_t0` ← first GENERIC type argument of `lv.v(int, UrtTimelineItem)`'s generic return type (`com.x.presenter.a<t0>` — a `ParameterizedType`, `actualTypeArguments[0]` is the real `com.x.urt.t0`); presenter_a's own sole abstract method return type is R8-erased to `java.lang.Object` and is only a bounds-resolved fallback (see Gotchas); `preroll_metadata` ← `getPrerollMetadata` return type. `resolver/Fingerprints.kt` holds 6 matchers (≥3 structural criteria each, obfuscated names never used as search criteria). Debug prefs: `hookforx_force_scan` (skip cache/known, pure scan — self-heal proof) and `hookforx_inject_bad_target` (fault-inject a wrong class to prove the validator's reject→fallback path), both written via MainActivity intent extras.
- `RepositoryHooker` — **the primary ad filter**, three verified layers:
  1. `com.x.repositories.urt.k.b(List, ...)` — shared list-processing entry, strips promoted items
  2. `com.x.repositories.urt.i$c$a.emit(Object, Continuation)` — Room→flow collector, home-timeline StateFlow injection point
  3. `com.x.urt.l.v(int, UrtTimelineItem)` — presenter factory; comment-section ads (promoMeta=true) are replaced with a **blank presenter** (runtime Proxy of `com.x.presenter.a` whose `c()` returns a no-op `com.x.urt.t0`; non-null so Compose LazyColumn doesn't crash). Detection: `getClientEventInfo().getComponent()` promoted/promo_ OR `getPromotedMetadata() != null` (via invokeOriginalMethod). Reflection Methods cached in a ConcurrentHashMap (hot path).
- `TimelineHooker` — `UrtTimelinePost.getPrerollMetadata()` → null (blocks video preroll ads; callers null-check it). View-layer fallback (`ViewGroup.addView` + RecyclerView `bindViewHolder`, obfuscated `RecyclerView$e0`) for classic non-Compose screens.
- `VideoAdHooker` — constructor counters on `PrerollMetadata`/`SspAdPodMetadata` (stats only).
- `PremiumProbeHooker` — **experimental** ad-relevant feature-flag / subscription-check probe. Default dump-only: DexKit-discovers X's boolean flag read points (`com.x.featureswitches` `(String, boolean)→boolean`) and subscription-state checks (`com.x.subscriptions`), hooks and logs unique ad-relevant keys once (`XLog.d`, throttled). `hook_premium_probe_override` (default off, opt-in) forces a single decompile-unambiguous key `feature/premium_plus` → true for A/B; if A/B shows no effect/breakage, empty `OVERRIDES` and keep dump-only (legal terminal state). Soft-fails cleanly when DexKit unavailable — never affects the three production hookers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usernameeve/HookforX](https://github.com/usernameeve/HookforX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
