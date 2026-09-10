---
trigger: always_on
description: Guidance for Claude Code when working in this repository. This project is a ground-up rebuild of **ES-DE Companion**, a dual-screen companion app for the ES-DE emulation frontend, using modern Android best practices.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository. This project is a ground-up rebuild of **ES-DE Companion**, a dual-screen companion app for the ES-DE emulation frontend, using modern Android best practices.

This document is deliberately weighted toward **structure, layering, and code quality** rather than a fixed feature spec. The core state pipeline is foundational and must be built to a higher standard of rigor than everything else in the app.

## The One Thing That Has To Be Rock Solid

Everything this app does depends on correctly understanding what ES-DE is currently doing. The pipeline is:

```
es_log.txt (raw text, growing/truncating file)
    → tail new lines
    → filter for "Scripting::fireEvent():" lines
    → parse into a typed Event
    → reduce events into a single current AppState
    → AppState drives everything downstream (media lookup, UI, widgets, whatever comes later)
```

This pipeline is the foundation everything else is built on. Treat it accordingly:

- It lives entirely in `domain`, as plain Kotlin with no Android framework dependencies where at all possible — the parsing and state-reduction logic should be testable with plain unit tests against fixture log lines, with no emulator, no instrumentation, no mocked Android classes.
- It should have the best test coverage of anything in the codebase. Every event type, malformed/unexpected line, empty-args case, and the truncate-on-restart case need explicit tests.
- Resist the urge to let UI or media-resolution concerns leak into this layer. Its only job is: raw log text in, correct `AppState` out. Nothing else should influence it, and it should not know anything about widgets, screens, or file paths for media.
- If this layer has a bug, the entire app is wrong regardless of how polished anything built on top of it is. Prioritize correctness and clarity here over cleverness or premature abstraction.
- **Cold start is not "replay the last event."** A freshly-launched app has no state to reduce onto, and context-dependent events (e.g. `screensaver-end`, which needs to know what was active *before* the screensaver started) produce the wrong result if the reducer only sees the single most recent line. `EsdeLogFileRepository` instead finds the last self-contained ("anchor") event and replays forward from there, growing its read window backward until it finds one or hits the start of the file. See `EsdeLogFileRepositoryTest` for the anchor-replay cases. That replay is also skipped if `es_log.txt`'s `lastModified()` predates the current boot — a device reboot can auto-start this app (it carries a `HOME` intent filter for secondary-display placement) before ES-DE has (re)written the log this session, so an anchor found in a not-yet-touched-since-boot file is guaranteed to be a leftover from before the reboot, not current truth.
- **The derived `AppState` stream is shared, not rebuilt per subscriber.** `ObserveAppStateUseCase` computes its reducer fold exactly once, at construction, as an application-scoped `StateFlow` — not fresh on every `invoke()` call. See Known Gotchas for why a naive per-call `scan()` silently produces stale/`Idle` state for late-joining subscribers.

## Architecture & Structure

**Clean Architecture, three layers, strictly one-directional dependencies:**

```
ui        → depends on → domain
data      → depends on → domain
domain    → depends on → nothing (no Android imports, no other layers)
```

- `domain`: models, the event parser, the state reducer, use-case classes, repository *interfaces*. Pure Kotlin. This is the layer that should be easiest to read, test, and reason about — it's the specification of what the app does, independent of how.
- `data`: repository *implementations* — file I/O, log tailing, media file resolution, persistence. Talks to Android/filesystem APIs. Implements the interfaces `domain` defines, never the reverse.
- `ui`: Compose screens + ViewModels. ViewModels depend on `domain` use cases/repository interfaces only — never directly on `data` implementations, never directly on `java.io.File`, `PackageManager`, or `FileObserver`.

Keep this a **single Gradle module** with package-based layering (`domain/`, `data/`, `ui/` packages) rather than splitting into multiple Gradle modules. Multi-module setups add build-config overhead that isn't justified at this project's size; package boundaries plus code review discipline are enough to keep the dependency rule honest. Revisit this only if the codebase grows enough that build times or team size actually demand it.

**Within each layer, favor small, single-purpose classes over large ones.** A class that parses log lines should not also decide file paths. A ViewModel for one screen should not accumulate responsibilities for another. If a class is hard to name in one sentence, it's probably doing too much.

## Code Quality & Maintainability Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RobZombie9043/es-de-companion](https://github.com/RobZombie9043/es-de-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
