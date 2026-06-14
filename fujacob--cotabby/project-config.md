---
trigger: always_on
description: Cotabby is a macOS menu bar app for on-device inline autocomplete. The core loop is:
---

# Cotabby Codex Instructions

## Project Identity

Cotabby is a macOS menu bar app for on-device inline autocomplete. The core loop is:

1. Track the currently focused editable field through Accessibility.
2. Monitor global keyboard input without stealing focus.
3. Decide whether the field, permissions, settings, and runtime are eligible.
4. Build an autocomplete request from the focused text context and optional visual context.
5. Generate locally through Apple Intelligence or llama.cpp.
6. Normalize the model output into a short continuation.
7. Render ghost text near the caret.
8. Insert accepted chunks when the user presses `Tab` while keeping the remaining tail alive.

Privacy and local-first behavior matter. Do not introduce hosted API dependencies unless the user
explicitly asks for that direction.

## Learning-First Collaboration

- Explain both the "what" and the "why" for architecture and code changes.
- Assume the user is actively learning Swift, AppKit, Accessibility APIs, llama.cpp integration,
  async/await, actor isolation, and macOS app architecture.
- Teach at the file, type, and subsystem level, not just the line level.
- Call out tradeoffs when there are multiple valid approaches.
- Prefer clean boundaries over quick coupling, especially across `App`, `UI`, `Services`, `Models`,
  and `Support`.

When creating or editing a file, explain:

- what the file is responsible for
- why the file exists as its own boundary
- which objects own it or collaborate with it
- how data flows into and out of it

When adding a `struct`, `class`, `enum`, actor, or protocol, explain:

- what responsibility it owns
- what other objects it collaborates with
- why it should exist as its own type instead of being folded into another file
- how long it lives and who owns it

## Repository Map

- `Cotabby/App/`: app entrypoint, composition root, lifecycle wiring, and coordinators.
- `Cotabby/UI/`: SwiftUI/AppKit presentation: settings, onboarding, menu views, overlays, and
  visual affordances.
- `Cotabby/Services/`: side-effectful boundaries: Accessibility, input monitoring, text insertion,
  screenshots/OCR, visual context, llama runtime, permissions, downloads, updates, and launch
  services.
- `Cotabby/Models/`: shared value types, settings snapshots, states, domain models, and protocol
  contracts.
- `Cotabby/Support/`: pure helper logic, prompt rendering, availability rules, normalization,
  reconciliation, geometry helpers, and low-level bridging utilities.
- `CotabbyTests/`: unit and microbench tests. Prefer testing pure `Support/` and `Models/` logic
  when possible.
- `CotabbyInference`: the llama.cpp wrapper, consumed as a SwiftPM package
  (`github.com/FuJacob/cotabbyinference`, pinned to `main`) rather than vendored in-tree.

## App Ownership

Start here when you need to understand lifecycle:

1. `Cotabby/App/Core/CotabbyApp.swift`
2. `Cotabby/App/Core/AppDelegate.swift`
3. `Cotabby/App/Core/CotabbyAppEnvironment.swift`

`CotabbyAppEnvironment` builds the long-lived dependency graph once. `AppDelegate` starts, stops,
and wires cross-subsystem subscriptions. SwiftUI views should observe objects from that graph
rather than creating services directly.

This ownership rule prevents duplicate Accessibility observers, duplicate input monitors, runtime
reload races, and mismatched settings state.

## Suggestion Pipeline

Read the coordinator in this order:

1. `Cotabby/App/Coordinators/SuggestionCoordinator.swift`
2. `Cotabby/App/Coordinators/SuggestionCoordinator+Lifecycle.swift`
3. `Cotabby/App/Coordinators/SuggestionCoordinator+Input.swift`
4. `Cotabby/App/Coordinators/SuggestionCoordinator+Prediction.swift`
5. `Cotabby/App/Coordinators/SuggestionCoordinator+Acceptance.swift`

The coordinator owns orchestration and user-facing state. It should not absorb every rule. Prefer:

- `SuggestionRequestFactory` for pure request construction
- `SuggestionAvailabilityEvaluator` for pure gating decisions
- `SuggestionSessionReconciler` for acceptance and active-tail reconciliation
- `SuggestionTextNormalizer` for backend-independent output cleanup
- `SuggestionWorkController` for generation task identity/cancellation
- `SuggestionInteractionState` for active suggestion session storage

This split matters because autocomplete is a state machine. Pure rules are easier to test and reason
about than coordinator mutations.

## Focus And Accessibility

Focus and geometry live in:

- `FocusTracker`: observes focus/value/selection changes and publishes snapshots.
- `FocusSnapshotResolver`: reduces raw AX elements into Cotabby-supported focus snapshots.
- `AXTextGeometryResolver`: resolves caret and input geometry.
- `AXHelper`: low-level Accessibility/Core Foundation helper calls.
- `FocusModels`: pure focus values, identities, capabilities, and debug inspection data.

Accessibility data is eventually consistent and app-specific. Browser editors, Electron apps,
native AppKit fields, and secure fields expose different AX shapes. Preserve stale-result guards,
`focusChangeSequence`, and capability checks unless the change explicitly replaces them.

## Visual Context And OCR

Visual context currently flows through:

- `VisualContextCoordinator`: field-scoped visual-context session lifecycle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FuJacob/cotabby](https://github.com/FuJacob/cotabby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
