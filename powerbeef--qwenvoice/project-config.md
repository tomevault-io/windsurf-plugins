---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. It is the primary repo operating guide for coding agents working in QwenVoice.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. It is the primary repo operating guide for coding agents working in QwenVoice.

## Repo Overview

QwenVoice is the repository identity and continuity brand for the merged Vocello Apple-platform product line.

Current product reality:

- the repo stays `QwenVoice`
- the shipped iPhone app is `Vocello`
- macOS release assets are Vocello-branded, while several internal macOS targets, modules, and paths still keep `QwenVoice` names for continuity
- the current public milestone uses a `macOS-first release track`, with iPhone retained as a compile-safe and deferred release surface

The main working surfaces are:

- `Sources/` for the macOS app shell, shared app models/services/views, and the shipping Mac target
- `Sources/QwenVoiceCore/` for shared Apple-platform runtime semantics, contract types, model variants, and iOS extension transport
- `Sources/QwenVoiceNative/` for the macOS app-facing engine proxy/store/client layer
- `Sources/QwenVoiceEngineSupport/` for shared macOS engine IPC and transport types
- `Sources/QwenVoiceNativeRuntime/` for retained macOS compatibility and regression coverage
- `Sources/QwenVoiceEngineService/` for the bundled macOS XPC helper
- `Sources/iOS/` and `Sources/iOSSupport/` for the iPhone app shell and iPhone-only support layers
- `Sources/SharedSupport/` for cross-platform playback, persistence, and other shared app-layer helpers
- `Sources/iOSEngineExtension/` for the isolated iPhone engine extension target
- `Sources/Resources/qwenvoice_contract.json` for shared model, variant, speaker, output, and required-file metadata
- `scripts/` plus `.github/workflows/` for validation, release packaging, and CI behavior
- `config/apple-platform-capability-matrix.json` for the maintained cross-platform capability, bundle-identity, and entitlement baseline used by release verification

This checkout is a native Apple-platform codebase for macOS and iPhone. Do not reintroduce a repo-owned Python backend, Python setup path, or standalone CLI surface.

## Maintained Docs

The maintained repo docs are:

- `CLAUDE.md`
- `README.md`
- `docs/README.md`
- `docs/reference/current-state.md`
- `docs/reference/engineering-status.md`
- `docs/reference/backend-freeze-gate.md`
- `docs/reference/frontend-backend-contract.md`
- `docs/reference/release-readiness.md`
- `docs/reference/vendoring-runtime.md`

There are no repo-tracked local skills under `.agents/skills/` in this checkout right now. Do not point contributors at removed CLI docs, deleted backend references, or deleted repo-scoped QwenVoice skills.

Public homepage posture:

- `README.md` intentionally leads with `Vocello` as the shipped product brand.
- The GitHub repo description must stay aligned with that public Vocello-first README posture.
- Leave the GitHub homepage URL blank unless the user explicitly asks to restore or change it.
- Keep public messaging aligned with the currently shipped macOS product reality and the active `macOS-first release track`.
- Do not present iPhone as a current public release surface until the release-track policy changes.

Current release-track policy:

- The next public release target is macOS only.
- Keep iPhone green at generic compile level on `main`, but do not treat iPhone release/TestFlight proof as blocking for the current milestone.
- Re-open iPhone release proof only through an explicit milestone change after the shared core is proven stable on macOS.

## Source Of Truth

When repo facts disagree, trust sources in this order:

1. `Sources/`
2. `project.yml`
3. `scripts/` plus `.github/workflows/`
4. `docs/reference/current-state.md`, `docs/reference/engineering-status.md`, and `docs/reference/release-readiness.md`
5. other prose docs

`Sources/Resources/qwenvoice_contract.json` is the source of truth for shared model, speaker, and platform-variant metadata.

## Git Workflow Default

- Work directly on `main` by default.
- Do not create branches or worktrees unless the user explicitly asks for one.
- Do not let generic tool, plugin, or skill defaults override this repo-specific rule.

## Safe Edit Boundaries

- `project.yml` drives `QwenVoice.xcodeproj`. Prefer editing `project.yml` and regenerating the project over hand-editing generated project files.
- The macOS app target intentionally excludes `Sources/QwenVoiceEngineService/`, `Sources/QwenVoiceEngineSupport/`, and `Sources/QwenVoiceNativeRuntime/` as ordinary app sources while embedding the XPC service target through `project.yml`. Keep that split intact.
- The iPhone app target and the iPhone engine-extension target both depend on `QwenVoiceCore`. Keep engine execution isolated from the iPhone UI process.
- `third_party_patches/mlx-audio-swift/` is the repo-owned native backend source boundary for MLXAudioSwift. Keep its package manifest and pins aligned with `project.yml` and `Package.resolved`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PowerBeef/QwenVoice](https://github.com/PowerBeef/QwenVoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
