---
trigger: always_on
description: This file defines repository-specific instructions for coding agents working on **InstallerX Revived**.
---

# AGENTS.md

## Purpose

This file defines repository-specific instructions for coding agents working on **InstallerX Revived**.

Use it to decide:

* where a change belongs,
* which constraints must be preserved,
* what to verify before claiming a task is complete.

Task-specific maintainer instructions take precedence over this file. When the request is narrow, make the smallest coherent change that satisfies it.

For substantial features, invasive refactors, or behavior changes that span several files, sketch a short implementation plan before editing. Keep the plan aligned with the actual implementation as the work proceeds.

---

## Read these first when relevant

* `README.md` — product scope, supported install flows, user-facing feature boundaries.
* `CONTRIBUTING.md` — translation policy, build prerequisites, contribution expectations.
* `.github/workflows/pr-check.yml` — the default CI build matrix used for pull requests.
* `settings.gradle.kts`, `app/build.gradle.kts`, and
  `gradle/libs.versions.toml` — before touching Gradle, repositories, flavors, versions, or dependencies.

Do not duplicate or contradict those files casually. Update this file only for stable, repository-wide rules that agents should repeatedly follow.

---

## Repository overview

InstallerX Revived is a community-maintained Android installer with:

* dialog, notification, and automatic installation flows,
* support for APK, APKS, APKM, XAPK, APKs inside ZIP files, and batch APK installation,
* profile-driven install options and install flags,
* privileged workflows involving Root, Shizuku, Dhizuku, and hidden APIs,
* switchable UI families based on Material 3 Expressive and Miuix.

Several product behaviors are intentionally flow-specific. Do **not** assume a feature supported in dialog installation is also valid for notification or automatic installation unless the existing code and docs already establish that.

---

## Critical project constraints

* Preserve the **online/offline** product boundary. The offline flavor must not silently gain network-only behavior or permissions.
* Prefer the repository’s existing **native API** paths and abstractions. Do not introduce shell-command implementations as a shortcut unless the maintainer explicitly requests it.
* Treat flow-specific behavior as flow-specific. A capability that exists for dialog installation is not automatically valid for notification or automatic installation.
* When changing behavior that is described in `README.md`, update it or call out the documentation impact in the handoff.

---

## Project layout

### Top-level areas

* `app/` — main Android application.
* `hidden-api/` — hidden API declarations/helpers consumed by the app.
* `build-plugins/` — shared Gradle convention plugins.
* `baselineprofile/` — Android baseline profile generation.
* `.github/workflows/` — CI and release automation.

Do not assume every top-level directory is an included Gradle module. Confirm active modules in
`settings.gradle.kts` before making module-level assumptions.

### Main Kotlin package map

Under `app/src/main/java/com/rosan/installer/`:

* `core/` — shared low-level app infrastructure.
* `data/` — persistence, concrete providers, repositories, and mappers.
* `di/` — Koin modules and initialization wiring.
* `domain/` — domain models, repository contracts, providers, use cases, and business rules.
* `framework/` — Android/platform-facing integration code.
* `ui/` — screens, widgets, navigation, themes, and UI-specific models.
* `util/` — utility helpers.

Preserve this separation. Do not move behavior into a convenient but wrong layer just to finish faster.

---

## Build prerequisites

### Toolchain

* Use the repository Gradle Wrapper: `./gradlew ...`.
* The project requires **JDK 25**.
* Kotlin/JVM toolchains and Android compile settings are centrally defined; do not downgrade or loosen them unless the task explicitly requires it.

### GitHub Packages authentication

The project resolves snapshot `miuix` artifacts from GitHub Packages.

For local builds, credentials are expected outside the repository, typically in the global Gradle properties file:

```properties
gpr.user=YOUR_GITHUB_USERNAME
gpr.key=YOUR_PERSONAL_ACCESS_TOKEN
```

The token needs `read:packages` access. CI may instead use `GITHUB_ACTOR` and `GITHUB_TOKEN`.

Never commit credentials, inline them into tracked files, or weaken the existing credential handling.

---

## Default verification

### Standard smoke build

For changes that do not affect connectivity flavors or variant-specific behavior, prefer the faster single-variant smoke build:

```bash
./gradlew assembleOnlineUnstableDebug \
  -PAPP_ID="com.rosan.installer.x.revived.test"
```

Use the full PR-CI pair when the change can reasonably affect app compilation across connectivity flavors, resources, dependency wiring, or variant-sensitive behavior:

```bash
./gradlew assembleOnlinePreviewDebug assembleOfflinePreviewDebug \
  -PAPP_ID="com.rosan.installer.x.revived.test"
```

### Report verification honestly

When summarizing work:

* state which commands were run,
* state whether they passed,
* say explicitly when verification was not run or could not be completed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wxxsfxyzm/InstallerX-Revived](https://github.com/wxxsfxyzm/InstallerX-Revived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
