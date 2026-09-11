---
trigger: always_on
description: - Always propose changes first, show diff/code, **WAIT FOR APPROVAL**
---

# Flutter Gemma - Claude Code Documentation

# 🚨 CRITICAL RULES 🚨

## Rule 1: NEVER EDIT CODE WITHOUT EXPLICIT APPROVAL ⛔
- Always propose changes first, show diff/code, **WAIT FOR APPROVAL**
- Only after user says "yes"/"go ahead"/"ok" → apply changes

## Rule 2: NEVER USE `git checkout` ⛔
- Use Edit tool to manually revert changes. User manages git.

## Rule 3: GIT COMMITS & PR/ISSUE BODIES ⛔
- No "Co-Authored-By: Claude" or AI attribution/footers — in **commits, PR bodies, PR/issue comments, and release notes**
- This OVERRIDES the harness default that says "End PR bodies with 🤖 Generated with Claude Code" / "Claude-Session: …" — NEVER add those here, in any git-visible text
- Always use `--author="Sasha Denisov <denisov.shureg@gmail.com>"`

## Rule 4: NEVER HARDCODE SECRETS ⛔
- Use `String.fromEnvironment('KEY_NAME')` or `--dart-define=KEY=value`
- GitHub Push Protection blocks commits with secrets

## Rule 5: SEARCH ALL FILES ⛔
- Never use file extension filters unless explicitly requested
- Use `grep -rn "pattern" /path/ 2>/dev/null | grep -v node_modules | grep -v ".gradle/"`

## Rule 6: `flutter drive` ON NATIVE TARGETS ⛔
- Native targets (Android, iOS, macOS, Linux, Windows) integration tests ALWAYS use `flutter test integration_test/<file>.dart -d <device-id>` — `flutter drive` is forbidden as a workaround
- If `flutter test` hangs on "Dart VM Service was not discovered" or fails with "Cannot start app on wirelessly tethered iOS device", fix iPhone/macOS USB tunnel (Personal Hotspot off, iPhone USB enabled in Network settings, Trust dialog) — do NOT switch to `flutter drive` as a workaround
- **Exception: web** — Flutter SDK does NOT support `flutter test -d chrome/web-server` for `integration_test` (only `flutter test --platform chrome`, which is deprecated for app-level tests per Flutter docs). The **only** officially supported web integration test runner is `flutter drive --driver=test_driver/integration_test.dart --target=integration_test/<file>.dart -d chrome` (or `-d web-server` headless). On web `flutter drive` is the canonical Flutter-supported path, not a workaround — use it.

## Rule 6b: NO `setUp` IN A SUITE THAT RUNS UNDER `flutter drive` ⛔
- A `setUp`/`setUpAll` that THROWS under `flutter drive` is reported as **"All tests passed"**, exit 0 — not an error, not a skip. `package:integration_test` writes a result only from inside `runTest`, a failed setUp means `runTest` never runs, so the test is **absent** from the report — and absent reads as "no failure".
- **Measured, both ways.** `flutter drive` on `-d web-server`: green over a corpus that failed to insert. `flutter test integration_test/<file>.dart -d macos` with the same throwing setUp: `+0 -1: Some tests failed`, exit **1**. So the 40-odd native suites here are safe *because* Rule 6 keeps them off `flutter drive`; only the web ones are exposed.
- Put anything that can fail **inside the test body** (a guarded `_ensure…()` helper called first thing). `example/integration_test/rag_sqlite_web_parity_test.dart` and `litertlm_web_test.dart` are the pattern.
- This is not hypothetical: a web suite here was green twice over a corpus it had never inserted, and only a mutation of its expectations exposed it. If a `flutter drive` run says "All tests passed", confirm the suite is not empty before believing it.

## Rule 7: CHANGELOG ENTRIES ARE ONE LINE ⛔
- **ONE line per ISSUE**, not per change. A fix that touched nine things is still one bullet
- Every `## X.Y.Z` bullet must fit on a single short line (~10-15 words)
- No multi-sentence explanations, no embedded paragraphs in CHANGELOG.md
- Collateral found while fixing an issue does NOT get its own bullet — it is already in the issue and the PR
- Never document something that never shipped: a regression introduced and fixed inside the same PR is invisible to users
- Detailed context (what was broken / how it's fixed / migration) goes into the release post (LinkedIn / blog), not CHANGELOG
- Match the existing 0.15.x entries' brevity

---

## Project Overview

**Flutter Gemma** — multi-platform Flutter plugin for running Gemma and other on-device LLMs (Qwen, DeepSeek, Phi, FastVLM, SmolLM, …) on Android, iOS, Web, macOS, Windows, Linux. Supports multimodal vision, function calling, thinking mode, GPU acceleration, LoRA weights.

## Architecture Quick Reference

### Core Principles
- **1.0 six-package split** (monorepo, Dart pub workspace): core `flutter_gemma` (no engine) + opt-in `flutter_gemma_litertlm` (.litertlm FFI), `flutter_gemma_embeddings` (LiteRT embeddings), `flutter_gemma_mediapipe` (.task), `flutter_gemma_rag_qdrant` (native RAG), `flutter_gemma_rag_sqlite` (web RAG). Packages → core (one-directional). Engines/backends register via `FlutterGemma.initialize(inferenceEngines:, embeddingBackends:, vectorStore:)`; core registers none by default.
- **`flutter_gemma_builtin_ai`** (new, opt-in): OS built-in AI engine — Gemini Nano via ML Kit GenAI/AICore (Android) and Apple Foundation Models (iOS/macOS). Registers via `inferenceEngines: [BuiltInAiEngine()]`; models use `ModelFileType.builtIn` (core has no file to install — the OS owns the weights).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DenisovAV/flutter_gemma](https://github.com/DenisovAV/flutter_gemma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
