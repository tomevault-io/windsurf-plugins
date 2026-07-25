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
| `clone-host/` | Host-side APK clone / identity reshape support library (compiled to a DEX asset). |
| `modules/` | Module Market catalog (`registry.json` + per-module folders). |
| `scripts/` | Build helpers and gates (`check_config_field_drift.py`). |

Runtime Kotlin is authored under `app/` and synced into `shell` by `syncShellRuntimeSources`. Edit the `app/` source once; do not permanently fork copies under shell.

User-facing product docs: `README.md`, `.github/docs/README_CN.md`, `.github/CONTRIBUTING.md`, `modules/README.md`.

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

- Host UI strings live in `app/src/main/java/com/webtoapp/core/i18n/Strings.kt` (split across `Strings` / `StringsA` … `StringsE`).
- Any new user-facing host string needs all 10 languages: Chinese, English, Arabic, Portuguese, Spanish, French, German, Russian, Japanese, Korean.
- Prefer adding properties on the existing split objects; match surrounding style.

## Android and packaging constraints

- Generated apps keep a low `targetSdk` (28) on the shell path because they rely on on-device fork+exec runtimes. Do not raise shell targetSdk casually.
- Avoid new third-party dependencies unless strongly justified (`app/build.gradle.kts` / `shell/build.gradle.kts`). Prefer platform APIs and existing modules.
- Notification push channels: Web Notification polyfill, polling, WebSocket, FCM (developer-owned Firebase config). Do not add OEM vendor push SDKs by default.
- Foreground services and notification helpers must use `SafeNotificationChannels` (or equivalent fail-soft create). Channel creation failures must not crash FGS startup.
- **One shell template:** `webview_shell.apk` from `:shell` release. Do not introduce a second template APK.
- Export incremental rebuild lives in `app/.../apkbuilder` (`ApkBuildCache` + `ApkBuilder`):
  - Modes: `FULL` / `CONTENT_OVERLAY` / `REUSE_UNSIGNED`.
  - Template / entry identities must be **content-stable** (no mtime-based keys).
  - Encrypted builds always force a full rebuild.
  - Do not feed signed or renamed APKs back into full `modifyApk` as templates.
- Port coordination: `PortManager` + `PortConflictMode` (`REASSIGN` / `AUTO_KILL` / `ALERT`) with real stop handlers. Local server runtimes must allocate through PortManager and clean up on stop.
- Local server / Linux env DNS: fork+exec runtimes (Node / PHP / Python / Go / WordPress / Linux env) should wire through `LocalDnsBridgeProxy` when they need host DNS/proxy bridging.
- Large runtime downloads use `NetworkModule.downloadClient` (extended timeouts), not the default short-lived client.
- HTML / FRONTEND packaged shells need file-scheme access via `ShellWebViewConfig` (`allowFileAccess` / local-file detection). Do not regress pure file-based HTML loads.
- Node.js export must embed `libnode_bridge.so`, `libnode.so` (16KB-aligned), and `libc++_shared.so` as native libs. Go export must embed `libgo_exec_loader.so`.
- Gradle custom tasks (`syncCloneHostDex`, etc.) must be configuration-cache safe: capture `File`/`Provider` values at configuration time, do not reference `Project`/`android.sdkDirectory` inside task closures.

## Workflow

- Do not commit secrets, `local.properties`, keystores, or IDE/cache junk.
- Do not create commits, push, open PRs, or file Issues unless the user asks to deliver / ship / push / open a PR (or equivalent).
- When changing export or shell packaging, rebuild the template you touched.
- When changing config fields, run `checkConfigFieldDrift` to catch model ↔ shell config name drift.

### Delivery (Issue + PR + CI)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiahonb777/web-to-app](https://github.com/shiahonb777/web-to-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
