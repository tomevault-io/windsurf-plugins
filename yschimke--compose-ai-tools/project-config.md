---
trigger: always_on
description: This file provides guidance to Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Agents when working with code in this repository.

## What this project is

A Gradle plugin (`ee.schimke.composeai.preview`) plus supporting tools that discover `@Preview` composables in compiled Kotlin classes and render them to PNG outside Android Studio. Targets both Jetpack Compose (Android, via Robolectric) and Compose Multiplatform Desktop (via `ImageComposeScene`).

## Documentation map: contributor vs. consumer

Two audiences, two doc trees. Don't conflate them:

- **This file + `docs/`** — contributor docs for working on *this repo*: editing the plugin, CLI, renderer modules, or VS Code extension; running the in-repo samples through `includeBuild("gradle-plugin")`; publishing releases. Build commands here use `./gradlew` against the local source tree.
- **[`yschimke/skills`](https://github.com/yschimke/skills)** — consumer docs for the *published* plugin and CLI live in a separate content repo. Two skill bundles:
  - [`compose-preview`](https://github.com/yschimke/skills/tree/main/skills/compose-preview) — applying `id("ee.schimke.composeai.preview")` to a downstream project and driving `compose-preview` against it. The `references/` subtree is per-target-stack and per-feature guidance:
    - [`permissions.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/permissions.md) — agent allowlists, staging PNGs under `build/`
    - [`state-hoisting.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/state-hoisting.md) — making composables previewable
    - [`capture-modes.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/capture-modes.md) — multi-preview annotations, paused-clock animations, `@ScrollingPreview`
    - [`a11y.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/a11y.md) — ATF accessibility checks
    - [`display-filters.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/display-filters.md) — post-process colour-matrix variants (bedtime grayscale, invert, daltonizer simulations)
    - [`agent-cloud.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/agent-cloud.md) — running in Claude/Codex/Gemini cloud environments (network allowlist, Setup script with `install.sh --android-sdk`, JVM-proxy gotcha)
    - [`cmp-shared.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/cmp-shared.md) — applying the plugin to a CMP `:shared` (`com.android.kotlin.multiplatform.library`) module: previews go in `commonMain`, JVM target gives the Desktop renderer something to attach to (issue #248)
    - [`wear-ui.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/wear-ui.md) — Material 3 Expressive design language for Wear OS
    - [`wear-tiles.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/wear-tiles.md) — Wear Tiles (protolayout-based, not Compose)
    - [`remote-compose.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/remote-compose.md) — Remote Compose (RemoteDocument byte stream for watch faces, tiles, widgets)
    - [`resource-previews.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/resource-previews.md) — Android XML resource captures (`<vector>`, `<adaptive-icon>`)
    - [`vscode.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview/references/vscode.md) — VS Code extension (humans, not agents)
  - [`compose-preview-review`](https://github.com/yschimke/skills/tree/main/skills/compose-preview-review) — sibling skill covering the PR-review surface: authoring agent-opened PRs, reviewing UI PRs locally (base + head render, diff, comment), and wiring `compose-preview/main` baselines + PR-comment GitHub Actions.
    - [`agent-pr.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview-review/references/agent-pr.md) — authoring agent-opened PRs and reviewing PRs opened by other agents
    - [`ci-previews.md`](https://github.com/yschimke/skills/blob/main/skills/compose-preview-review/references/ci-previews.md) — maintaining a `compose-preview/main` branch with rendered PNGs and a `baselines.json` for diff-on-PR workflows

The bootstrap installer's canonical home is now [`yschimke/skills/scripts/install.sh`](https://github.com/yschimke/skills/blob/main/scripts/install.sh); it pulls both skill bundles from `yschimke/skills` (default `main`) and the CLI tarball from this repo's releases. A thin curl-pipe stub remains at this repo's `scripts/install.sh` so old snippets keep working. When you change consumer-facing behaviour (a new flag, a network requirement, a setup-script step), edit `yschimke/skills` — both the SKILL.md and the installer live there. Cross-link from here when contributors need the same information for sandbox setup (e.g. the Android SDK bootstrap referenced from "Bringing up a fresh sandbox" below).

## Common commands

Build / test everything:
```
./gradlew check                   # plugin unit + functional tests, CLI tests
```

Render the sample previews (end-to-end smoke test of the full pipeline):
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yschimke/compose-ai-tools](https://github.com/yschimke/compose-ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
