---
trigger: always_on
description: Biscotti is a native **macOS meeting recorder**: it records meeting audio (mic + system), produces on-device diarized transcripts, integrates with the calendar, and lives mostly in a menu-bar app. Private, local, Apple-silicon-only (macOS 15+).
---

# Biscotti — Repo Guide

Biscotti is a native **macOS meeting recorder**: it records meeting audio (mic + system), produces on-device diarized transcripts, integrates with the calendar, and lives mostly in a menu-bar app. Private, local, Apple-silicon-only (macOS 15+).

**Current stage:** Scaffolding (Project 0) and the **Stage A foundations** are built. Beyond the scaffolding (`BiscottiKit`, `App/`, `Makefile`, `hooks-mcp`, CI, lint/format), the repo now has the `Transcription` and `AudioCapture` packages, the `DataStore` + `ManualTestKit` modules in `BiscottiKit`, the `ManualTestApp` (XcodeGen) that hosts the shared `XPCServices/BiscottiTranscriber.xpc` and `XPCServices/BiscottiLLM.xpc`, and a manual-test CI gate — all green on `lint`/`test`/`build_app`. The `LocalLLM` package (`Packages/LocalLLM`) has been graduated from `experiments/llm` with a production NSXPC transport (`BiscottiLLM.xpc`) and a `Local LLM` ManualTestApp tab. The **`llm_features` spec project** has built the app-level AI features: the `Intelligence` module in `BiscottiKit` (LLM-powered summarization + speaker-name inference), wired into `AppCore` (auto-run after transcription), `MeetingDetailUI` (Summary tab, speaker names, mapping sheet), and `SettingsUI` (AI Enhancements section with model download). **The Stage A foundations are complete**, including the on-hardware manual-test pass (the non-gating `manual-tests-check` job is now green). The next *product* step is the MVP (Record → Transcribe). See the roadmap (`specs/implementation_plan.md`).

> This file is a map. Read the specific docs below before acting; don't rely on this summary alone for decisions.

---

## Read this first, by what you're doing

- **Anything at all / new to the repo** → `specs/app_overview.md` (the product) + this file.
- **Designing or building a component, or asking "where does X live / what depends on it?"** → `specs/architecture.md`.
- **Deciding what to build next, scoping a new `/spec` project, or ordering work** → `specs/implementation_plan.md`.
- **Touching audio / calendar / transcription / permissions** → the matching `specs/research/<area>/README.md` (validated decisions — don't re-derive them) **and** the corresponding `experiments/<Name>/` (working reference code).
- **Understanding *why* the design is shaped this way / the rules it follows** → `specs/projects/library_design/functional_spec.md`.

---

## Key files and how they relate

The docs form a chain, each feeding the next:

```
specs/app_overview.md  ─►  specs/research/**  ─►  experiments/**
  (what to build)            (how — proven)         (reference code)
        │                         │                       │
        └─────────────────────────┴───────► specs/architecture.md ─► specs/implementation_plan.md
                                              (where it lives)         (order to build it)
```

### Product & vision (the "what" and "why")
- **`specs/app_overview.md`** — the master product spec. Every feature, the UX intent, "Misc App Reqs," design style, and the stack/testing philosophy (Swift-package-first, thin app target). **Source of truth for product intent.** Read when you need to know what a feature is supposed to do.
- **`specs/plan.md`** — the high-level staging (Research → Scaffolding → Library Building → App). Short; gives the big-picture sequence. `specs/implementation_plan.md` is the detailed version of this.

### The design — master roadmap (the "where" and "in what order") — under `specs/`
- **`specs/architecture.md`** — the **static topology**: every component (package vs. module vs. app-glue), its responsibilities (as outcomes, not interfaces), dependency DAG, granularity rationale, thin-app composition, cross-cutting conventions, and P2/P3 placement. **Deliberately shape-level — it designs *no* concrete interfaces/schemas.** Read before building or modifying any component to know its home, boundaries, and dependencies.
- **`specs/implementation_plan.md`** — the **build roadmap**: an ordered list of ~14 Projects (each a future `/spec new project`), with a contents list, dependencies, and risk per Project — but no internal phases (those are decided when each Project is spec'd). Read to pick/scope the next Project. Project 0 (Scaffolding) is the first.

These two were authored in the `library_design` spec project and promoted under `specs/` as the durable, living roadmap. Keep them updated as the build progresses.

### Validated research (the "how" — already proven, don't re-litigate)
- **`specs/research/README.md`** — one-page summary of all technical decisions (audio API, format, models, isolation, permissions, distribution) with a recommendations table. Start here for any technical area.
- **`specs/research/audio/README.md`** — audio capture/recording decisions. Plus `specs/research/audio/phase9_validation_findings.md` (what changed after real-hardware validation — e.g. global capture, route-change survival, ADTS AAC) and `specs/research/audio/meeting_app_bundle_ids.md` (seed watchlist data).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scosman/Biscotti](https://github.com/scosman/Biscotti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
