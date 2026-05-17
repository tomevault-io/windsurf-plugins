---
trigger: always_on
description: This repository is optimized for:
---

# AGENTS.md

## Purpose

This repository is optimized for:
- fast iteration
- AI-assisted development
- low-context edits
- reliable shipping

Favor:
- simple code
- explicit structure
- local reasoning
- predictable patterns

Avoid:
- over-engineering
- premature abstraction
- architectural churn

---

# Structure

```txt
App/                  — window lifecycle, AppDelegate
Core/                 — shared types and cross-feature contracts only
Features/
  Notch/              — notch UI and settings panel
  Cursor/             — cursor companion, long-press, click hooks
  Context/            — screenshot capture, OCR, Gemini, memory
  Agent/              — Sonnet wiring, computer-use harness
  Onboarding/         — first-launch permission prompts
vendored/             — read-only reference code (do not edit, do not include in target)
```

---

# Feature Layout

Each feature owns its code. Keep related code together.

```txt
Features/Notch/
├── NotchContentView.swift      — root; open/closed (420×280); Home/Settings tabs; Cmd+D + swipe; tab persisted via @AppStorage
├── NotchHomeView.swift         — Home tab: orb, transcript, activity log (logs completion entry on run done)
├── AgentSettingsView.swift     — Settings tab: 4 knobs + Advanced section (system prompt, context diagnostics)
├── ClosedNotchView.swift       — resting dot states in closed notch
├── NotchShape.swift            — custom Shape for notch geometry
└── AgentStateView.swift        — standalone status row (available, not in tabs)

Features/Cursor/
├── CursorCompanion.swift       — coordinator; implements CursorAppearanceSetting
├── CursorCompanionView.swift   — SwiftUI PNG sprite
├── CursorCompanionViewModel.swift
├── CursorCompanionWindow.swift — transparent always-on-top NSPanel
├── CursorTracker.swift         — tracks real cursor position
├── LongPressDetector.swift     — fires .longPressBegan / .longPressEnded
└── LongPressEvents.swift       — notification name constants

Features/Context/
├── ContextCoordinator.swift    — entry point; implements RecentActivityContext
├── ContextClickMonitor.swift   — debounced click hook (Accessibility API)
├── ContextSnapshotStore.swift  — rolling buffer of screenshots (max 20)
├── ContextMemoryStore.swift    — learned UI memory on disk
├── ContextOCRService.swift     — native OCR via Vision framework
├── ContextGeminiObservationService.swift
├── ContextGeminiObservationModels.swift
├── ContextActivationBuilder.swift  — buffer → compact prompt packet
├── ContextMemoryRenderer.swift
├── ContextModels.swift
├── ContextWindowMetadataReader.swift
├── ContextTextSignalFilter.swift
├── ContextAIObservationLog.swift   — in-memory Gemini event log + ContextGeminiObservationGate (rate limiter)
├── ContextDevToolsWindowController.swift — separate Dev Tools window for telemetry (Cmd+Option+D)
├── ContextDebugView.swift          — Dev Tools console: pause/resume gathering, overview, injected packet, captures/OCR, Gemini I/O, learned memory, metrics
└── ContextPerformanceReporter.swift

Features/Agent/
├── VoiceRecordingService.swift — records mic on .longPressBegan; runs WhisperKit (whisper-tiny) on .longPressEnded; posts .transcriptReady
├── AgentSession.swift          — subscribes to .transcriptReady; reads lastTranscript; fires one harness turn
├── ComputerUseHarness.swift    — multi-turn Claude computer-use loop (model: claude-sonnet-4-6)
├── ComputerUseModels.swift     — Codable API types
├── AnthropicClient.swift       — URLSession API client
├── ToolDispatcher.swift        — tool calls → CGEvent actions; handles all computer-use actions incl. F-keys + emoji
└── AgentRunMetrics.swift       — per-run metrics logging

Features/Onboarding/
├── OnboardingView.swift        — three permission cards
├── OnboardingWindowController.swift
└── PermissionChecker.swift     — live permission polling
```

Do not create:
- global managers
- giant shared services
- generic utility dumping grounds

---

# Architecture

Preferred flow:

```txt
View
 ↕
ViewModel (if needed)
 ↕
Service / Actor
```

| Layer | Responsibility |
|---|---|
| View | rendering + user interaction |
| ViewModel | UI state + orchestration |
| Service/Actor | API calls, storage, OS side effects |
| Models | lightweight data types |

---

# Rules

## 1. Prefer Locality

If code is only used by one feature, keep it inside that feature. Do not abstract early.

---

## 2. Keep Dependencies Simple

Allowed:

```txt
Feature → Core
```

Avoid:
- Feature → Feature imports
- circular dependencies
- hidden shared state

Shared logic belongs in `Core/`.

---

## 3. Keep Files Focused

Target: ~100–500 LOC, one primary responsibility. Split when reasoning becomes difficult.

---

## 4. Use Explicit Names

Prefer:

```swift
AgentSession
ContextCoordinator
CursorCompanion
```

Avoid:

```swift
Manager
Helper
Utils
BaseObject
```

Names should be searchable and unambiguous.

---

## 5. Prefer Modern Swift

Use:
- SwiftUI
- async/await
- `actor` for shared mutable state across async contexts
- `@MainActor` on `ObservableObject` singletons
- structs + value semantics for models

Avoid:
- unnecessary protocols
- deep inheritance
- DispatchQueue.main.async (use `await MainActor.run` or `@MainActor` instead)

---

## 6. Cross-Feature Contracts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sam-siavoshian/agent-notch](https://github.com/sam-siavoshian/agent-notch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
