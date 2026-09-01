---
trigger: always_on
description: This document is the LeapFlow engineering collaboration contract. It is not only a style guide: it defines the design, runtime, UX, safety, and verification rules that every implementation change must follow.
---

# AGENTS.md

This document is the LeapFlow engineering collaboration contract. It is not only a style guide: it defines the design, runtime, UX, safety, and verification rules that every implementation change must follow.

## Design Philosophy

1. **Signal-Driven Intelligence** — All agent intelligence derives from observing real-world signals, not from hardcoded rules. If a behavior cannot be learned from signals, it is not in scope.

2. **Context Pipeline as Core** — Signal → Filter (SNR) → Compress (intent-preserving) → Store (multi-layer) → Retrieve (goal-dependent) → Decide. Every feature and every external signal source, including IM collaboration events, must map to this pipeline before it can drive action.

3. **Progressive Trust** — Never auto-execute on first encounter. Earn autonomy through repeated success: DRAFT → CANDIDATE → VERIFIED → PRODUCTION.

4. **Occam's Razor** — The simplest correct solution wins. Reject complexity that doesn't directly serve user value. Every abstraction must pay for itself.

5. **LLM-Native Design** — Design for LLM reasoning first. Protocols over classes. Declarative over imperative. Context over configuration.

6. **User-Centric Reliability** — User experience is part of correctness. Every change must keep common paths easy, predictable, recoverable, and must not degrade adjacent workflows.

## Code Quality Requirements

- SOLID principles are non-negotiable; implementations must be cohesive, well-factored, and easy to reason about
- Occam's Razor: maximize elegance and efficiency, reject unnecessary complexity
- Design for generalization and universality; prefer reusable domain concepts over one-off special cases
- Easy to extend, avoid hardcoding and hard rules
- Industrial-grade robustness: every external call has timeout, retry, and fallback
- Structured error propagation: failures must flow as typed envelopes (`FailureEnvelope`) with source, category, recoverability, and side-effect state — never as ad-hoc dicts, bare strings, or unclassified exceptions
- User experience is a first-class quality bar: optimize for clarity, ease of use, fast feedback, and graceful recovery
- All comments and docstrings in English
- Type annotations on all public APIs
- No bare except — always specify exception types

## Architecture Principles

- **System Boundary Awareness**: LeapFlow is a multi-entry, multi-module runtime. Changes must account for the affected path across CLI/TUI, leapd, engine, skills/tools, LLM, storage, memory, gateway, hub, and platform adapters.
- **TUI as the Primary User Entry**: The interactive TUI is the default product surface. Preserve streaming feedback, command queue behavior, approval prompts, status bar accuracy, long-input robustness, history, and session continuity.
- **Concurrent TUI Instances Are a Supported Scenario (MANDATORY)**: several TUIs in *different workspaces*, sharing one leapd and one profile, is a normal way to use LeapFlow — not an edge case. Each instance must remain fully usable and must see only its own session, conversation, context usage, and turn state. A change to session routing, `status()`, stream metadata, the client lease, or anything the status bar renders is not verified until it has been exercised with two instances in two workspaces at the same time. One instance degrading another is a release blocker, not a limitation to document.
- **TUI Command Clarity**: Global task-control commands stay short and unambiguous (`/cancel`, `/skip`, `/pause`, `/resume`, `/queue`, `/drop`); teach-mode controls must use the `/teach ...` namespace and should not keep bare compatibility aliases during early iteration.
- **TUI Prompt Ownership**: Input prompt and placeholder rendering must have a single owner. Avoid duplicate prompt sources; placeholder text stays visually subordinate, offset after the prompt, and disappears as soon as the user types.
- **leapd Runtime Consistency**: Daemon-backed behavior must preserve lifecycle correctness: start, stop, restart, status, RPC streaming, cancellation, pending approvals, runtime config reload, multi-client state, and version consistency.
- **Progressive Context Disclosure (PCD)**: Keep one unified execution loop, but never default every turn to full disclosure. Each LLM call must use the smallest sufficient PromptAssemblyPlan for tools, memory, history, reasoning, streaming, and risk; upgrade progressively only when observable signals require it.
- **Gateway as Signal Boundary**: External IM/platform integrations are not just messaging features; they extend LeapFlow's Observe/Orient boundary into collaboration environments. Inbound platform events must enter as structured signals (`BackendEvent` → normalized domain event/message), pass SNR filtering and privacy/safety gates, then feed memory, decision, and action paths according to their classification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelscope/leapflow](https://github.com/modelscope/leapflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
