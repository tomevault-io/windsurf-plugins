---
trigger: always_on
description: Instructions for coding agents working in this repository.
---

# AGENTS.md

Instructions for coding agents working in this repository.

## Code style

- Do what you believe is right. Make the change complete and correct, not the smallest possible diff. If a fix calls for refactoring, renaming, or touching multiple files, do it.
- Match the patterns and conventions already in the surrounding code.
- Do not add copyright or license headers unless asked.

## Project layout

| Path | Role |
|------|------|
| `app/` | Full builder host: editor UI, export pipeline, runtimes, preview. |
| `shell/` | Runtime template. Built to `app/src/main/assets/template/webview_shell.apk` via `:shell:assembleRelease` + `:app:syncShellTemplateApk`. |
| `clone-host/` | Host-side APK clone / identity reshape support library. Its DEX asset generation (`syncCloneHostDex`) is deliberately disabled (`enabled = false`, AV false-positive mitigation, e0d2d4d6) — `AppCloner` runs fail-soft without the asset. |
| `modules/` | Module Market catalog (`registry.json` + per-module folders). |
| `sample-bundles/` | Heavy sample dependency packs (`python-*-shared.zip` + sha256-pinned `manifest.json`) fetched on demand by `SampleSharedPackManager` — deliberately NOT in `app/assets` (saves ~30MB raw / ~7MB compressed from the host APK). Regenerate via `scripts/build_sample_bundles.py`. |
| `docs/` | VitePress documentation site (guide / developer / extensions, EN + ZH), published to https://shiaho777.github.io/web-to-app/ by `.github/workflows/docs-deploy.yml`. Site URL paths map 1:1 to files under `docs/` (`/zh/...` → `docs/zh/...`). |
| `scripts/` | Build helpers and gates (`check_config_field_drift.py`). |

Runtime Kotlin is authored under `app/` and synced into `shell` by `syncShellRuntimeSources`. Edit the `app/` source once; do not permanently fork copies under shell.

User-facing product docs: `README.md`, `.github/docs/README_CN.md`, `.github/CONTRIBUTING.md`, `modules/README.md`. The published documentation site is https://shiaho777.github.io/web-to-app/ (source: `docs/`, deployed by `.github/workflows/docs-deploy.yml`).

## How the main pieces connect

```text
Editor (Compose screens in app/)
  ↔ data models (WebApp, configs)
  ↔ export factory (ApkConfig / ApkConfigJsonFactory)
  ↔ ApkBuilder / ApkBuildCache  →  signed generated APK

app/ sources
  → syncShellRuntimeSources  →  shell DEX  →  webview_shell.apk (template)

Generated APK runtime
  WebToAppApplication → ShellModeManager → load assets JSON config
  → WebViewManager / runtime servers (Node/PHP/Python/Go/WordPress)
```

Mental model:

- **Host preview** runs `:app` with all classes on the main classpath.
- **Generated APK** runs the shell template classes (full runtime synced from `app/`), reading config from assets JSON via `ShellModeManager`.
- A flag in the editor is useless at export unless it flows through **model → ApkConfig JSON → shell config → runtime code**.

## i18n

- Host UI strings live under `app/src/main/java/com/webtoapp/core/i18n/` (facade `object Strings` in `Strings.kt`, delegating to `StringsA` … `StringsE`, one file per split object).
- **All** user-visible strings must be inline `when (Strings.lang)` blocks covering all 10 languages: Chinese, English, Arabic, Portuguese, Spanish, French, German, Russian, Japanese, Korean. `when(lang)` blocks may never use `else ->` — `AppStringsResourceConsistencyTest` and `StringsKtTranslationParityTest` enforce this.
- **Never** load user-visible text via `context.getString(R.string.*)` / `stringResource(R.string.*)`. `res/values/strings.xml` holds only `translatable="false"` resources (e.g. `app_name`) and no locale `values-*/` directories exist, so a localized resource string could never cover the 10 languages and would silently fall back to the default `values/` (Chinese). Use `Strings.xxx` (or `Strings.funName(arg)` for parameterised strings — see `linuxEnvInstalledToast(name)` for the pattern). Tests gate this: `kotlin source never references R string for user-visible text`, plus `values strings xml only holds non-localised resources` and `no locale values dirs or grouped app strings files exist` which block resurrecting resource-based strings.
- `R.string` is reserved for `translatable="false"` non-localised resources only (e.g. `app_name`).
- Prefer adding properties on the existing split objects (`StringsA` … `StringsE`, one object per file); match surrounding style.
- **Shell gets generated string subsets, not the synced files.** `syncShellRuntimeSources` excludes `core/i18n/Strings*.kt`; `generateShellStrings` (→ `scripts/generate_shell_strings.py`) scans the synced runtime sources for `Strings.x` / `StringsX.y` references (aliases `val S = Strings` handled) and emits reduced `Strings.kt` / `StringsA-E.kt` under `shell/build/generated/shellStrings` carrying only referenced members plus the facade infrastructure — editor-only strings never reach the shell template or generated APKs (~0.75 MB compressed saved per APK). Missing references fail the shell compile loudly; nothing fails silently. Author strings in `app/` exactly as before — no extra step needed.

## Android and packaging constraints

- Generated apps keep a low `targetSdk` (28) on the shell path because they rely on on-device fork+exec runtimes. Do not raise shell targetSdk casually.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiaho777/web-to-app](https://github.com/shiaho777/web-to-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
