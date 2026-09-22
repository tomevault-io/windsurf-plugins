---
trigger: always_on
description: This file defines the architecture, coding conventions, and required workflows for the `zero_inspector_kit` Flutter plugin. AI coding agents (CodeBuddy, Trae, Cursor, Claude Code, GitHub Copilot, Codex, etc.) should read and follow it for any task in this repository. It is the single source of truth for project conventions.
---

# Zero Inspector Kit - Agent Guide

This file defines the architecture, coding conventions, and required workflows for the `zero_inspector_kit` Flutter plugin. AI coding agents (CodeBuddy, Trae, Cursor, Claude Code, GitHub Copilot, Codex, etc.) should read and follow it for any task in this repository. It is the single source of truth for project conventions.

## Overview
`zero_inspector_kit` is a Flutter plugin providing an in-app developer console: network inspection, logging, database viewing, memory monitoring, FPS monitoring, and route tracking for Android and iOS. Integration is one line (`ZeroInspectorKit.runAppWithInspector`). The inspector auto-disables in release builds (tree-shaken out).

## When to apply
- Implementing features, fixing bugs, or changing the public API in `lib/`.
- Opening pull requests (titles must follow Conventional Commits; 3 status checks are required to merge).
- Cutting a release or publishing to pub.dev.
- Updating the GitHub Pages documentation site.
- Editing `pubspec.yaml`, the workflows under `.github/workflows/`, or the native Android/iOS code.

## Architecture
- `lib/zero_inspector_kit.dart` - public barrel; re-export the public API only. Add new public symbols here.
- `lib/zero_inspector_kit_platform_interface.dart` - `ZeroInspectorKitPlatform` abstract base.
- `lib/zero_inspector_kit_method_channel.dart` - default `MethodChannel` implementation.
- `lib/zero_inspector_kit_dio.dart` - Dio integration helper.
- `lib/src/` - 33 implementation files grouped as: `interceptors/` (dio, http, log, route_observer), `platform/` (platform_channel), `models/` (database_info, interceptor_rule, leak_record, log_entry, memory_snapshot, network_request, route_entry), `services/` (database_provider, database_service, export_service, fps_service, inspector_service, memory_inspector_service, sqlite_provider), `utils/` (environment, inspector_log, memory_leak_tracking), `ui/` (conditional_inspector, database_viewer, floating_button, fps_viewer, inspector_panel, log_viewer, memory_trend_chart, memory_viewer, network_viewer, route_viewer, theme/inspector_theme). Do not import `lib/src/` directly from consumers.
- Public API entry: the `ZeroInspectorKit` class (in the barrel) exposes `init()`, `wrapApp()`, and `runAppWithInspector()`. The barrel also re-exports the interceptors, services, UI widgets, and utils listed above so consumers use them via the package root, never via `lib/src/`.

## Dependencies and SDK constraints
- Dart SDK: `>=3.11.0 <4.0.0`; Flutter: `>=3.3.0` (from `pubspec.yaml`).
- Runtime deps: `plugin_platform_interface`, `http`, `sqflite`, `path_provider`, `collection`. Keep the caret (`^`) constraint on pub dependencies; do not pin exact versions without reason.
- Dev deps: `flutter_test`, `flutter_lints` (v6). Analysis is governed by `analysis_options.yaml`.
- License: MPL-2.0 (Mozilla Public License 2.0). Do not relicense without the maintainer's explicit decision.
- Platform pattern: define the abstract API in the platform interface, provide the `MethodChannel` default, register it in the barrel.
- How features work: network capture via `HttpOverrides` (covers `http` and Dio's `HttpClient`); logging via Zone plus `debugPrint` override; memory/FPS via VM Service plus `addTimingsCallback` (v1.2.1+ uses real frame timestamps and `rasterFinish - buildStart` duration to catch GPU jank).
- Native: Android `android/src/main/kotlin/.../ZeroInspectorKitPlugin.kt` (package `com.zerolabsco.zero_inspector_kit`); iOS `ios/Classes/ZeroInspectorKitPlugin.swift`. Keep native changes minimal and matching the method channel contract.

## Coding conventions
- Follow `effective_dart`; style is enforced by `flutter analyze` / `flutter_lints` (v6) in CI.
- Use Conventional Commits for both commit messages AND PR titles: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, `build:`, `ci:`, `chore:`, `revert:`.
- Do not break the public API without a major version bump.

## Workflows

### Branching and PRs
- Branch from `main` with a typed prefix: `feat/`, `fix/`, `docs/`, `ci/`, `chore/`, etc.
- Never push directly to `main`; it is branch-protected and requires 3 passing status checks to merge.
- PR titles MUST follow Conventional Commits, enforced by `pr-title-check.yml` (`amannn/action-semantic-pull-request@v6`).
- Required checks before merge: `Analyze & Test`, `Pana Score Check`, `Check PR Title (Conventional Commits)`.

### Pull request body template / PR 正文模板

AI coding agents (CodeBuddy, Trae, Cursor, Claude Code, GitHub Copilot, Codex, etc.) and contributors SHOULD follow the body template below when opening PRs. Keep the `###` section structure; fill in real content. Use English as the primary language and Chinese as the secondary language (EN-primary, ZH-secondary) for each section. Brand the assistant with **Zero Buddy** (two words, NOT "ZeroBuddy") at the end.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zero-labsco/zero_inspector_kit](https://github.com/zero-labsco/zero_inspector_kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
