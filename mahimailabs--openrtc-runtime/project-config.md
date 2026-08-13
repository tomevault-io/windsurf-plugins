---
trigger: always_on
description: This repository is a Python open-source package built on LiveKit for voice AI applications. The package should be production-minded, easy to understand, well-typed, testable, and friendly for contributors.
---

# AGENTS.md

## Purpose
This repository is a Python open-source package built on LiveKit for voice AI applications. The package should be production-minded, easy to understand, well-typed, testable, and friendly for contributors.

When making changes, prioritize:
1. Correctness and reliability in real-time voice flows
2. Clear public APIs
3. Backward compatibility for open-source users
4. Strong typing and test coverage
5. Minimal, maintainable abstractions

---

## Product and Architecture Principles

### Core goals
- Build reusable Python components for voice AI on LiveKit
- Keep framework-specific details isolated where possible
- Make real-time behavior predictable and observable
- Prefer explicit APIs over magic or hidden behavior
- Optimize for contributor readability, not cleverness

### Architectural rules
- Keep business logic separate from transport/runtime glue
- Keep LiveKit integration code isolated from domain logic
- Avoid tightly coupling high-level package APIs to internal implementation details
- Prefer composition over inheritance
- Keep modules focused and small
- Do not introduce global mutable state unless absolutely necessary
- Avoid singleton-heavy designs

### Layering
Use this mental model when organizing code:
- `api/` or public package surface: stable interfaces intended for users
- `core/`: domain logic, orchestration, state handling
- `integrations/` or `livekit/`: LiveKit-specific adapters and runtime hooks
- `models/` or `types/`: typed shared schemas, events, config objects
- `utils/`: narrow helper functions only, no hidden business logic

Business logic must not live inside:
- CLI entrypoints
- example scripts
- callbacks that should delegate into core logic
- transport adapters

---

## Code Style

### General Python style
- Target modern Python syntax supported by the project version
- Use type hints everywhere
- Prefer explicit return types on public functions and methods
- Prefer small pure functions where practical
- Prefer `dataclass` or clear typed classes over loose dictionaries for structured data
- Prefer `Enum` for constrained option sets
- Avoid boolean flag arguments when an enum or separate function would be clearer
- Avoid overly terse variable names
- Avoid surprising side effects

### Readability
- Write code for maintainers and contributors unfamiliar with the internals
- Favor straightforward control flow over clever compact code
- Add comments only when the why is not obvious from the code
- Do not add noisy comments that restate the code
- Keep functions focused on one responsibility
- If a function needs extensive explanation, split it into smaller helpers

### Naming
- Use `snake_case` for variables, functions, and modules
- Use `PascalCase` for classes
- Use `UPPER_SNAKE_CASE` for constants
- Name functions after what they do, not how they do it
- Use domain-specific names such as `session`, `turn`, `utterance`, `participant`, `track`, `transcript`, `agent`, `pipeline`, `vad`, `stt`, `tts` when appropriate

### Imports
- Prefer absolute imports within the package
- Keep imports grouped and sorted
- Avoid circular imports by improving module boundaries rather than using late imports unless necessary
- Do not introduce heavy dependencies in core modules unless justified

---

## Public API Guidelines

### Stability
- Treat public APIs as stable unless the task explicitly allows a breaking change
- Avoid renaming or reshaping public interfaces without strong justification
- If a breaking change is necessary, update docs, changelog notes, and tests

### API design
- Public APIs should be easy to discover and hard to misuse
- Prefer explicit configuration objects over long argument lists
- Prefer sensible defaults
- Validate user input early with clear error messages
- Raise precise exceptions with actionable messages
- Avoid leaking internal implementation details through public return values

### Async design
- Use async only where it is justified by I/O or runtime integration
- Keep async boundaries clean and intentional
- Do not mix sync and async styles inconsistently in the same API surface
- Avoid blocking operations in async code
- Use cancellation-safe patterns where relevant

---

## LiveKit-Specific Guidance

### Real-time constraints
- Real-time audio paths should avoid unnecessary allocations, blocking calls, and hidden latency
- Be careful with backpressure, task buildup, and event storms
- Time-sensitive paths should be simple and observable
- Do not add logging noise in hot loops

### Event-driven behavior
- Make event handling explicit
- Prefer well-defined event payload types over loosely shaped dicts
- Guard against race conditions in session lifecycle logic
- Be careful around connect/disconnect, participant joins/leaves, track subscription changes, and stream interruptions

### Voice pipeline concerns
- Preserve clear boundaries between:
  - audio input handling
  - VAD / turn detection
  - STT
  - LLM or agent reasoning
  - TTS
  - playback/output
- Avoid coupling one stage tightly to another unless necessary
- Keep pipeline stages mockable for tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahimailabs/openrtc-runtime](https://github.com/mahimailabs/openrtc-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
