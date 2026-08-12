---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

Reference plugins for [CodeOnTheGo](https://github.com/appdevforall/CodeOnTheGo) (CoGo / CotG). Each top-level folder (`Beepy/`, `apk-viewer/`, `markdown-preview/`, `keystore-generator/`, `snippets/`, `random-xkcd/`, `icons-repository/`, `ndk-installer-plugin/`, `sketch-to-ui-plugin/`) is an independent Gradle project that builds a `.cgp` plugin installable via the CoGo Plugin Manager. They're held together only by the shared `libs/` jars at the repo root.

## Common commands

Build one plugin:

```sh
cd Beepy   # or any plugin folder
./gradlew assemblePlugin           # release .cgp -> build/plugin/<pluginName>.cgp
./gradlew assemblePluginDebug      # debug variant
```

Build every plugin from scratch (after rebuilding libs):

```sh
./scripts/update-libs.sh                          # default: github.com/appdevforall/CodeOnTheGo@stage
./scripts/update-libs.sh --ref <branch-or-tag>
./scripts/update-libs.sh --local ../CodeOnTheGo   # use an existing checkout instead of cloning
```

The script clones CoGo into `.cache/CodeOnTheGo/` on first run, rebuilds both jars, copies them into `libs/`, then runs `assemblePlugin` for every example. It auto-detects examples by scanning for `build.gradle.kts` files that apply `com.itsaky.androidide.plugins.build`.

`local.properties` must contain `sdk.dir=...`. The committed `local.properties` at the repo root is harmless leftover; each plugin needs its own.

## Git workflow

- **Always work on a branch.** Never commit directly to `main` — branch first (`git switch -c ...`) even for a one-line fix. Working on `main` is almost never right for this repo.
- **Fetch before you diff against `main`.** Any time you compute or reason about a diff against `main` (code review, PR base, "what changed"), run `git fetch origin` first and compare against `origin/main`. A stale local `main` produces phantom findings — a `/code-review` here once flagged 3 issues that were outside the actual PR diff because local `main` was ~50 files behind `origin/main`. When a diff-against-main is requested, suggest fetching first.

## Architecture

### `libs/` is the load-bearing piece

Every plugin depends on two jars in the repo-root `libs/`:

- **`plugin-api.jar`** — the IDE-side API surface (`IPlugin`, `PluginContext`, `BuildStatusListener`, `IdeBuildService`, etc.). Each plugin uses it as `compileOnly` (provided by the IDE at runtime) AND as `buildscript classpath` so the Gradle plugin can resolve symbols at configuration time.
- **`gradle-plugin.jar`** — the Gradle plugin with id `com.itsaky.androidide.plugins.build`, applied by every plugin. It's the output of CoGo's `plugin-api/plugin-builder/` module (separate from CoGo's `gradle-plugin/` module, which is unrelated despite the name). It packages the compiled Android library into a `.cgp`.

There is also **one shared Gradle wrapper at the repo root** (`gradlew` + `gradle/wrapper/`). New plugins should use it — build them with `cd <plugin> && ../gradlew assemblePlugin` rather than bundling a per-plugin `gradlew`/`gradle/wrapper/` copy. (`flutter-template` follows this; most older plugins still carry their own local wrapper and can be migrated opportunistically.)

Both jars are referenced via `../libs/*.jar`. **Always use the repo-root `libs/` jars and the repo-root Gradle wrapper — never bundle per-plugin copies.** A plugin that ships its own `libs/plugin-api.jar` / `libs/gradle-plugin.jar` (e.g. copied from another plugin) can drift out of sync with the rest of the repo; point `build.gradle.kts` (`compileOnly`) and `settings.gradle.kts` (buildscript `classpath`) at `../libs/*.jar` and delete any local `libs/`. The root `plugin-api.jar` already carries the full API surface (including `IdeTemplateService`/`CgtTemplateBuilder`), so newer sub-APIs do not justify a local copy. **A plugin folder is not standalone in isolation** — copy the root `libs/` along if you move one elsewhere. When CoGo's API changes, refresh via the script above or the **Update libs from CodeOnTheGo** GitHub Action (which also commits the refreshed jars, cuts a release, and deploys `.cgp` files to the website).

### Plugin shape

A plugin is an Android *application* module (despite installing as a library) with:

1. **`build.gradle.kts`** applies `com.android.application`, `org.jetbrains.kotlin.android`, and `com.itsaky.androidide.plugins.build`. Configures `pluginBuilder { pluginName = "..." }`. Uses `compileOnly(files("../libs/plugin-api.jar"))` — never `implementation`.
2. **`settings.gradle.kts`** declares the two jars on the buildscript classpath plus AGP and Kotlin.
3. **`src/main/AndroidManifest.xml`** declares plugin identity as `<meta-data>` entries on `<application>`: `plugin.id`, `plugin.name`, `plugin.version` (resolved from `${pluginVersion}`), `plugin.description`, `plugin.author`, `plugin.main_class`, `plugin.min_ide_version`, and optional `plugin.permissions`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appdevforall/plugin-examples](https://github.com/appdevforall/plugin-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
