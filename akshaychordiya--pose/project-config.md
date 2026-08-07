---
trigger: always_on
description: Guidance for AI agents (Claude Code, Cursor, Codex, etc.) working on the Pose codebase. Read this before you touch anything.
---

# AGENTS.md

Guidance for AI agents (Claude Code, Cursor, Codex, etc.) working on the Pose codebase. Read this before you touch anything.

## What Pose is

A KSP2 processor that auto-generates Jetpack Compose `@Preview` functions from `@Composable` source at build time. Deterministic, offline, never modifies user source. Ships with a companion IntelliJ plugin for editor-side gutter navigation.

Full user-facing docs: [`README.md`](README.md). This file is for people (and agents) *working on* Pose.

## Repository layout

| Path                 | What lives here                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `annotations/`       | Pure Kotlin JVM annotations JAR - `@Pose`, `@PoseProvider`, `@PoseIgnore`. Zero Compose deps by design. Published to Central.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `processor/`         | The KSP2 symbol processor. Reads `@Pose`, emits `<Source>__Preview.kt` files. Also published to Central.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `intellij-plugin/`   | The Marketplace plugin. Adds gutter icons in the IDE that jump from source composables to their generated previews.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `sample-app/`        | End-to-end smoke test. Every capability of Pose is exercised in one file each ([LoginContent](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/LoginContent.kt), [HomeContent](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/HomeContent.kt), [ProductCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/ProductCard.kt), [ArticleCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/ArticleCard.kt), [WellKnownTypesCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/WellKnownTypesCard.kt), [SelfThemedBanner](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/SelfThemedBanner.kt)). If a change plausibly affects generation, verify it here too. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AkshayChordiya/Pose](https://github.com/AkshayChordiya/Pose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
