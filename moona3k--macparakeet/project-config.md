---
trigger: always_on
description: > Context for AI coding assistants working on MacParakeet.
---

# CLAUDE.md

> Context for AI coding assistants working on MacParakeet.

## What is MacParakeet?

A **fast, private, local-first voice app** for macOS with three co-equal modes: system-wide dictation, file transcription, and meeting recording. Powered by NVIDIA's Parakeet TDT via FluidAudio CoreML on the Neural Engine.

**North Star:** Fast, local-first voice app for Mac.

**Domain:** [macparakeet.com](https://macparakeet.com)

**Pricing:** Free and open-source (GPL-3.0)

## Quick Navigation

| Need | Go To |
|------|-------|
| What are we building? | `spec/README.md` -> spec index and roadmap |
| Product vision | `spec/00-vision.md` |
| Data model | `spec/01-data-model.md` |
| Feature details | `spec/02-features.md` |
| Architecture | `spec/03-architecture.md` |
| UI patterns | `spec/04-ui-patterns.md` |
| Audio pipeline | `spec/05-audio-pipeline.md` |
| STT engine | `spec/06-stt-engine.md` |
| Text processing | `spec/07-text-processing.md` |
| Error handling | `spec/08-error-handling.md` |
| Testing strategy | `spec/09-testing.md` |
| AI coding methodology | `spec/10-ai-coding-method.md` |
| LLM integration | `spec/11-llm-integration.md` |
| Processing layer (prompts, actions, workflows) | `spec/12-processing-layer.md` |
| ADRs (locked decisions) | `spec/adr/` -> individual decision records |
| CLI testing guide | `docs/cli-testing.md` |
| Brand identity | `docs/brand-identity.md` |
| UI/UX design overhaul | `docs/design-overhaul.md` |
| Distribution, signing & auto-updates | `docs/distribution.md` |
| Telemetry system | `docs/telemetry.md` |
| Commit message format | `docs/commit-guidelines.md` |
| Implementation plans | `plans/` -> active and completed plans |

## Tech Stack (Locked Decisions)

| Layer | Choice | Notes |
|-------|--------|-------|
| Platform | macOS 14.2+ | Apple Silicon only |
| Language | Swift 6.0 | SwiftUI for UI |
| Database | SQLite | GRDB (single file, dictation history + transcriptions + meeting recordings) |
| STT | Parakeet TDT 0.6B-v3 | Via FluidAudio CoreML/ANE (~2.5% WER, 155x realtime, 25 European languages) |
| Audio | AVAudioEngine + Core Audio + Core Audio Taps | Mic capture for dictation; Core Audio Taps for system audio (meeting recording); FFmpeg (bundled) for video file conversion |
| YouTube | yt-dlp | Standalone macOS binary, weekly non-blocking auto-update via `--update` |
| Auto-Update | Sparkle 2 | In-app updates via EdDSA-signed appcast (non-App Store) |

## Product Context

MacParakeet is extracted from the OatFlow feature in Oatmeal but is maintained independently -- no shared packages, no monorepo dependencies.

| | MacParakeet | Oatmeal |
|---|-------------|---------|
| **Focus** | Voice dictation + file transcription + meeting recording | Meeting memory + calendar |
| **Complexity** | Simple, focused | Complex, powerful |
| **Pricing** | Free (GPL-3.0) | Freemium + Pro |
| **Value prop** | "Fast local transcription" | "Remembers everything" |

## Product Decisions (Settled)

These decisions were made during spec review and are locked:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Empty transcript UX | Silently dismiss | Short hold-to-talk with no speech = user changed their mind. No error card. |
| Audio retention | On/off toggle | Simpler than 3-tier (all/7d/never). Users who care about storage can manually delete. |
| Processing mode scope | Global default only | Set once in Vocabulary, applies to all dictations. No per-dictation picker on overlay. |
| Context awareness | Aspirational future | No version commitment. Don't promise what doesn't exist. Build post-launch. |

## Architecture Decisions (ADRs)

All ADRs are in `spec/adr/`. These are locked decisions -- don't second-guess them.

| ADR | Decision | File |
|-----|----------|------|
| ADR-001 | Parakeet TDT 0.6B-v3 as primary STT | `spec/adr/001-parakeet-stt.md` |
| ADR-002 | Local-first processing (amended: opt-in LLM providers, telemetry) | `spec/adr/002-local-only.md` |
| ADR-004 | Deterministic text processing pipeline | `spec/adr/004-deterministic-pipeline.md` |
| ADR-005 | First-run onboarding flow | `spec/adr/005-onboarding-first-run.md` |
| ADR-007 | FluidAudio CoreML migration (Python elimination) | `spec/adr/007-fluidaudio-coreml-migration.md` |
| ADR-009 | Custom hotkey support (any single key + chord combos) | `spec/adr/009-custom-hotkey.md` |
| ADR-010 | Speaker diarization via FluidAudio offline pipeline | `spec/adr/010-speaker-diarization.md` |
| ADR-011 | LLM via cloud API keys + optional local providers | `spec/adr/011-llm-cloud-and-local-providers.md` |
| ADR-012 | Self-hosted telemetry via Cloudflare (Worker + D1) | `spec/adr/012-telemetry-system.md` |
| ADR-013 | Prompt Library + multi-summary architecture | `spec/adr/013-prompt-library-multi-summary.md` |
| ADR-014 | Meeting recording via Core Audio Taps | `spec/adr/014-meeting-recording.md` |
| ADR-015 | Concurrent dictation and meeting recording | `spec/adr/015-concurrent-dictation-meeting.md` |
| ADR-016 | Centralized STT runtime and two-slot scheduler | `spec/adr/016-centralized-stt-runtime-scheduler.md` |
| ADR-017 | Calendar-driven meeting auto-start (proposed) | `spec/adr/017-calendar-meeting-auto-start.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moona3k/macparakeet](https://github.com/moona3k/macparakeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
