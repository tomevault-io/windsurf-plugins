---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

kslides is a Kotlin DSL for the [reveal.js](https://revealjs.com) presentation framework. Presentations are authored in Markdown, HTML, or the Kotlin HTML DSL (kotlinx.html). Output can be static HTML files (for Netlify/GitHub Pages) or a dynamic HTTP server (Ktor, for Heroku).

## Build Commands

```bash
./gradlew build -x test     # Build without tests
./gradlew test              # Run tests (Kotest + JUnit 5)
./gradlew buildFatJar       # Build executable fat JAR (kslides-examples)
./gradlew clean             # Clean build artifacts
./gradlew stage             # Heroku deployment build
./gradlew lintKotlin        # Lint with Kotlinter
./gradlew formatKotlin      # Auto-format with Kotlinter
./gradlew detekt            # Static analysis with Detekt
./gradlew dependencyUpdates # Check for dependency updates
```

Run a single test class:
```bash
./gradlew :kslides-core:test --tests "com.kslides.PresentationTest"
```

Run the example presentation locally:
```bash
java -jar build/libs/kslides.jar
```

### Makefile Shortcuts

Common wrappers in `Makefile`:

```bash
make help                  # list all targets (default target)
make build                 # clean + gradle build -x test
make lint                  # lintKotlinMain + lintKotlinTest
make detekt                # Detekt static analysis (fails on findings)
make tests                 # cleanTest test
make uber                  # fatjar + run the example jar
make versions              # dependencyUpdates
make check-site            # uv lock --upgrade --dry-run for the docs site
make upgrade-site          # uv lock --upgrade for the docs site
make site                  # clean-site + serve the Zensical docs site
make upgrade-wrapper       # upgrade the Gradle wrapper to the libs.versions.toml version
make publish-local         # publishToMavenLocal
make publish-snapshot      # publish -SNAPSHOT to Maven Central (signed)
make publish-maven-central # release to Maven Central (signed)
```

The `publish-snapshot` and `publish-maven-central` targets sign via `GPG_ENV`, which exports three vanniktech-maven-publish env vars: `ORG_GRADLE_PROJECT_signingInMemoryKey` (armored secret key from `gpg --armor --export-secret-keys $GPG_SIGNING_KEY_ID`), `ORG_GRADLE_PROJECT_signingInMemoryKeyId` (the same key id, needed when a subkey is selected), and `ORG_GRADLE_PROJECT_signingInMemoryKeyPassword` (read from the macOS Keychain via `security find-generic-password -a gpg-signing -s gradle-signing-password`). These publish targets only work on macOS with those credentials configured. Publishing uses the [vanniktech `maven-publish`](https://github.com/vanniktech/gradle-maven-publish-plugin) plugin; signing only runs when `signingInMemoryKey` is set, so `make publish-local` and snapshot builds work without the GPG env.

### CI

`.github/workflows/ci.yml` runs on PRs to `master` and on pushes to `master`. Expect green CI before merging.

`.github/workflows/docs.yml` builds the Zensical docs site under `website/kslides/` plus the Dokka HTML and publishes them to GitHub Pages. The published layout is: root → Zensical site, `/api-docs/` → Dokka HTML, `/docs/` → example slides.

### Releasing

The current release is `1.1.1` (tag `1.1.1`, GitHub release `v1.1.1`, published to Maven Central as `com.kslides:kslides-core` and `com.kslides:kslides-letsplot`); `1.0.0` was the first stable tag. To cut a new release: bump `version` in `gradle.properties`, update `CHANGELOG.md`, `RELEASE_NOTES.md`, `README.md`, `llms.txt`, and the docs site (`website/kslides/docs/installation.md`), run `make publish-maven-central`, then create a GitHub release whose tag matches the version (no `v` prefix on the tag, `v` prefix on the title).

## Module Structure

Three Gradle modules defined in `settings.gradle.kts`:

- **kslides-core** — Core DSL library: slide types, configuration, page rendering, Ktor server, filesystem output. This is what consumers depend on.
- **kslides-examples** — Example presentations with `main()` entry point in `Slides.kt`. Uses ShadowJar to build `kslides.jar`. Main class: `SlidesKt`.
- **kslides-letsplot** — Lets-Plot visualization integration (JetBrains Lets-Plot). Depends on kslides-core.

### Build conventions

Shared build logic lives in `buildSrc/` as two precompiled-script convention plugins:

- `kslides.kotlin-module` — applies `kotlin("jvm")`, the JVM toolchain (read from `libs.versions.toml` via the `jvm` version key), Kotlinter, [Detekt](https://detekt.dev) (group `dev.detekt`, plugin id `dev.detekt`), the `kslides.stable-versions` convention plugin (which applies [Ben-Manes versions](https://github.com/ben-manes/gradle-versions-plugin) and rejects non-stable `dependencyUpdates` candidates), and the kotest/logback test dependencies. Detekt fails the build on findings by default (the config is valid and the tree is violation-free); pass `-Pdetekt.ignoreFailures=true` to downgrade to report-only while iterating. Honors `-PoverrideVersion=...` so snapshot builds can override the gradle.properties version.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kslides/kslides](https://github.com/kslides/kslides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
