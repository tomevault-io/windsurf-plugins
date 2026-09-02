---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Project

Org Mobile — an Android viewer/editor for Emacs org-mode files (`com.alsciokat.orgmobile`). The org files live in a user-picked SAF folder and are synced by an external program; the app must never corrupt or reformat what it doesn't explicitly edit.

## Commands

```bash
./gradlew :core:org:test                  # parser/timestamp/edit tests (pure JVM)
./gradlew :core:data:testDebugUnitTest    # vault/agenda/search/reminder tests
./gradlew assembleDebug                   # build APK
./gradlew :core:org:test --tests '*OrgEditsTest*'   # single test class

adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.alsciokat.orgmobile/.MainActivity

cd web && npm run build          # editor bundle + themes -> app/src/main/assets/
cd web && npm test               # doc-engine unit tests; npm run test:browser for the bundle

# Convert an Emacs theme into a theme CSS file (see "Themes"), then rebuild the assets.
tools/emacs-theme-to-css.py ~/.emacs.d/elpa/modus-themes-*/modus-vivendi-theme.el \
    -o web/themes/modus-vivendi.css
```

Sample corpus for manual testing is in `sample-org/`; push with
`adb push sample-org /sdcard/Documents/org-sample`. The user's real vault is `/sdcard/Documents/org`.

## Commits

Write commit messages with a concise one-line imperative summary, followed by a blank line
and a bulleted list of only the core changes. Use the minimum number of bullets warranted by
the commit—one bullet is preferred when there is only one core change. Do not pad the list
with implementation details, verification steps, or separate restatements of the same change.

Before committing, update this AGENTS.md so its documentation matches the change—whenever the
commit adds, removes, or alters behavior described here (architecture, data flow, invariants,
toolchain, UI behavior), revise the relevant prose in the same commit. Keep the doc and the
code in sync; a stale description is a defect.

## Toolchain constraints (hard-won — do not "fix" these)

- SDK root is `/opt/android-sdk` (see `local.properties`). Platform packages are minor-versioned (`platforms;android-37.1` — there is no plain `android-37`).
- Gradle runs on **JDK 21** via `org.gradle.java.home` in `gradle.properties`; the system Java (26) is too new for AGP.
- **AGP 9 has built-in Kotlin**: applying `org.jetbrains.kotlin.android` to an Android module is a hard error. The `org.jetbrains.kotlin.plugin.compose` plugin is still required for Compose. The pure-JVM `:core:org` module uses `kotlin("jvm")` normally.
- Glance: `actionStartActivity<T>()` (reified) does not exist in glance-appwidget — use the `Intent` overload. Glance `Text` does not auto-size; widget font sizes are hard-coded sp values. Widget picker previews are **static** `android:previewLayout` XML (`res/layout/widget_*_preview.xml`) — Glance widgets render blank in the picker otherwise; keep the preview roughly in sync with the real layout by hand. Default size is set via `targetCellWidth`/`targetCellHeight` with `minWidth`/`minHeight` following Android's `70·cells − 30` dp formula (agenda widget is 3×2).
- **Android's ICU regex engine** is stricter than the JVM's: a bare `}` in a pattern crashes at runtime (`PatternSyntaxException`) though it compiles and passes JVM unit tests. Always escape `\}`; avoid `[\s\S]` (use `RegexOption.DOT_MATCHES_ALL`).
- After reinstalling the app, Samsung's launcher shows "Couldn't add widget" on existing widget instances until the app process pushes fresh content — launch the app once (or wait for `SyncWorker`). Not a bug; don't chase it.
- WebDAV uses **OkHttp** (`:core:data`; `mockwebserver` for tests), image loading uses **coil-network-okhttp**, and secrets use **androidx.security:security-crypto** (`EncryptedSharedPreferences`, still the standard despite maintenance mode). The manifest needs `INTERNET` and sets `android:usesCleartextTraffic="true"` (WebDAV is user-configured and often a self-hosted LAN box over plain `http://192.168.x.x`; Android blocks cleartext by default on API 28+, failing both OkHttp and Coil). WebDAV multistatus XML uses `javax.xml`; keep any `mobileorg.org`/index regex ICU-safe (escape `\}`, no `[\s\S]`).

## Architecture

```
:core:org    pure-JVM org parser/model/edits + MobileOrg protocol — no Android deps
:core:data   vault (SAF + WebDAV file access, cache), agenda, search, reminders — Android lib but logic is JVM-testable via FakeOrgFileStore
:app         Compose UI, notifications, widgets, settings, vault registry
```

```
Architecture Diagram:

+-------------------+                   +-------------------+                   +-------------------------+                 +-----------------------+
| /app              |                   | /app/.../data     |                   | /core/.../data          |                 | /core/.../orgmode     |
| AppRoot -> UI     | ---- access ----> | VaultManager      | ---- manage ----> | OrgVault                | --- contain --> | OrgDocument           |
|                   |               |-> | VaultActions      | ---- modify ---/  |                         |  `--- use ----> | OrgParser             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alsciokat/org-mobile](https://github.com/alsciokat/org-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
