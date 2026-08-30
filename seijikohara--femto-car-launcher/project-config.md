---
trigger: always_on
description: Android home launcher for in-car displays across three device
---

# Femto Car Launcher

Android home launcher for in-car displays across three device
classes — aftermarket CarPlay / Android Auto AI boxes, built-in
Android head units, and car-mounted smartphones. MVP targets Android 13 (API 33).

It is a regular Play-Store Android app installed on those devices — **not** an
OEM-embedded (Android Automotive) system app, and **not** an Android Auto /
CarPlay projection app (the "built-in Android head units" are aftermarket Android
units, not the car's factory system). In-car visibility and operability are the
design priority, but a **safe default the user can override** (e.g. the UI-scale
setting), not a hard mandate — see #automotive-overrides.

<!-- "multi-region distribution" is prose-cited by NominatimApi.kt. -->
The launcher is designed for **multi-region distribution**. No
single market is privileged in design, code, or documentation;
locale-specific behaviour is parameterised, and the strictest
applicable rule wins when markets diverge.

> **Rule locations.** AGENTS.md is the tool-agnostic project brief
> and rule SSOT for every coding agent (cite rules here as
> `AGENTS.md#<anchor>`); path-scoped rules live in
> `.claude/rules/*.md` (cite by file path; rule-file anchors are not
> addressable from outside). One home per rule; link, never restate.
> Tool-specific surface (Claude Code agents, skills, memory) lives in
> `CLAUDE.md`, which imports AGENTS.md.

## Tech stack <a id="tech-stack"></a>

- Kotlin (auto-applied by AGP), Jetpack Compose (via the BOM),
  Material 3; JDK 21 toolchain, Java 11 source/target. Versions:
  `gradle/libs.versions.toml` + `gradle/wrapper/gradle-wrapper.properties`
  (the JDK toolchain version itself is pinned in
  `gradle/gradle-daemon-jvm.properties`).
- `minSdk = 33`, `targetSdk = 36` with `compileSdk { release(37) }`
  (compile against API 37 as `androidx.core` 1.19+ requires; the
  supported-device floor stays Android 13 / API 33).
- Web map page (`webmap/`): TypeScript (native TS 7 compiler) +
  Vite+ (the `vp` CLI: build / test / oxlint / oxfmt) + `maplibre-gl` /
  `mapbox-gl`, managed with pnpm (pinned via `packageManager`). The
  Vite `build.target` is `chrome109` — the Android 13 factory WebView;
  aftermarket AI boxes may never update it, so never raise it
  without revisiting that floor (phone WebViews stay current, but
  the strictest device class governs). Rules: `.claude/rules/webmap.md`.

## Source layout

```
app/src/
├── main/
│   ├── java/io/github/seijikohara/femto/
│   │   ├── MainActivity.kt           # ComponentActivity, single launcher entry
│   │   ├── data/                     # One sub-package per domain (apps, calendar, clock, common, diagnostics, display, dock, fonts, geocoding, location, music, system, voice, weather); data/ never imports ui/
│   │   └── ui/
│   │       ├── <area>/               # One area per top-level surface
│   │       │   ├── <Area>Route.kt    # VM-binding entry point (when stateful)
│   │       │   ├── <Area>Screen.kt   # Pure UI; takes UiState + onAction
│   │       │   ├── <Area>UiState.kt  # data class + Action sealed (when stateful)
│   │       │   ├── <Area>ViewModel.kt# StateFlow<UiState>; handles Action
│   │       │   └── components/       # Area-private widgets
│   │       └── theme/                # FemtoTheme + tokens + PreviewLightDark
│   └── res/                          # themes (values{,-night}/), strings (per-locale once locales are wired up), icon drawables, xml/
├── test/...                          # JVM unit tests (runTest + TestDispatcher)
└── androidTest/...                   # Compose UI tests (createComposeRule)
```

`webmap/` (top level) is the TypeScript source of the LIVE map
WebView page; Gradle builds it into `assets/web/` via the
node-gradle plugin (`node {}` in `app/build.gradle.kts` is the
wiring SSOT; nothing under `src/main/assets/web/` is committed).
`gradle/libs.versions.toml` is the dependency catalog SSOT
(webmap npm deps: `webmap/package.json` + lockfile).

Trivial stateless screens need only `<Area>Screen.kt` — see
`.claude/rules/compose.md`.

## Rules

### Automotive overrides <a id="automotive-overrides"></a>

| Concern | M3 default | Femto rule | Symbol |
| --- | --- | --- | --- |
| Tap target | 48 dp | **≥ 64 dp** | `FemtoDimens.MinTouchTarget` |
| Body text on the head-unit dashboard | flexible | **≥ 16 sp** — the body floor sits on the rem-style type scale rooted at `FemtoDimens.BaseTextSize` (16 sp), from which every size derives; never `bodySmall` / `labelSmall`. Cards may deliberately relax this for glance metadata, metrics, progress captions, and dense reference text (e.g. license/log listings) — never as a literal in component code: the size lives in `FemtoDimens.GlanceTextSize` (12 sp) or inside a named `Type.kt` extension (e.g. `cardMeta`, `monoReference`). `ui/home/components/`, `ui/licenses/`, and `ui/diagnostics/` are the reference for where the relaxation applies (inherited from the retired dashboard-v2 mockup, whose KDoc notes mark each spot) | `FemtoDimens.MinBodyTextSize` / `FemtoDimens.GlanceTextSize` |

When the value lives in code, the symbol on the right is the SSOT —
not a magic number in another file.

These floors are the **safe default** (the `MEDIUM` UI scale), not a hard ceiling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seijikohara/femto-car-launcher](https://github.com/seijikohara/femto-car-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
