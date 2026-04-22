---
trigger: always_on
description: Repository-level collaboration and engineering rules for AI agents and developers.
---

# AGENTS.md

## Purpose
Repository-level collaboration and engineering rules for AI agents and developers.

## First-Read Order (AI-First)
1. `/Users/sunqin/study/language/cpp/code/MFCMouseEffect/AGENTS.md`
2. `/Users/sunqin/study/language/cpp/code/MFCMouseEffect/docs/agent-context/current.md`
3. `/Users/sunqin/study/language/cpp/code/MFCMouseEffect/docs/refactoring/phase-roadmap-macos-m1-status.md`
4. Targeted docs for the touched capability (single issue/refactoring doc, not bulk-read; prefer starting from `/Users/sunqin/study/language/cpp/code/MFCMouseEffect/docs/agent-context/p2-capability-index.md`)

## Global Collaboration Rules
1. Keep each file focused (single responsibility). Split logic instead of continuously inflating old files.
2. Prefer low-coupling architecture and reusable boundaries. Avoid patch-only fixes unless emergency.
3. Do not break existing architecture for short-term gains. Propose major architecture changes first (benefit/cost/risk/migration), then wait for review.
4. After changes, run relevant builds/tests by yourself before handing over.
5. Every behavior/interface change must sync docs under `/Users/sunqin/study/language/cpp/code/MFCMouseEffect/docs` in the same iteration.
6. Small-step delivery: split commits by single capability; include scope, risks, validation, and docs.
7. If accumulated changes span many capability points/files, propose and execute staged commits to control regression/review cost.

## Collaboration Split (Macro/Micro)
- User owns macro direction:
  - architecture boundaries
  - design tradeoffs
  - priorities and acceptance criteria
- Codex owns micro execution loop:
  - implementation/refactor/debug
  - local diagnostics collection + analysis
  - regression validation + docs/index synchronization

## Issue Triage Contract
Before modifying behavior for reported anomalies, classify first:
- `Design behavior`
- `Bug/regression`
- `Insufficient evidence`

Include short evidence (code path/config/runtime behavior). If user-visible behavior changes, explain the change and get confirmation first.

## Debug/Testing Collaboration Contract
1. User does minimal reproduction actions only; agent handles logs/diagnostics collection and analysis.
2. Prefer local ignored debug artifacts; avoid temporary interfaces.
3. For executable/file-lock failures before build/package, try self-recovery first (terminate stale process, retry).
4. If user says tested/no issue, treat as pass and continue. Only branch into debugging when user reports issue.
5. New timing/threshold/frequency features must support production + test values with explicit boundary (setting/env/debug entry) and docs.

## Platform and Tooling Notes
### Current host priority
- Current development priority: macOS mainline first.
- Keep Windows behavior regression-free.
- Linux follows compile-level + contract-level unless explicitly expanded.

### macOS <-> Windows source sync workflow
- This repo currently uses `Syncthing` for automatic source sync between the macOS main workspace and the Windows validation workspace.
- Default collaboration assumption:
  - macOS workspace is the primary development source
  - Windows workspace is used for build / run / visual validation
  - do not tell the user to manually copy the latest code to Windows when the task is part of the normal sync workflow
- Default Windows path assumption for commands:
  - `F:\language\cpp\code\MFCMouseEffect`
- When giving Windows manual commands, prefer the synced Windows workspace path directly instead of a generic placeholder path unless the user asks otherwise.
- Treat sync-specific issues as workflow issues first:
  - if Windows sees unexpected local changes, conflict files, or failed items, consider Syncthing state / ignore rules / receive-only behavior before blaming code changes
  - do not ask the user to manually re-copy files as the first fallback when the synced workspace should already contain the latest sources
- Windows command execution:
  - Prefer direct SSH execution from macOS for Windows-side build, packaging, log collection, and environment checks.
  - Do not maintain a synced manual handoff file for normal Windows steps; use direct SSH first, and only ask the user for manual action when remote execution is unavailable or the task genuinely requires UI interaction.

### macOS native stack evolution (Decision: 2026-02-27)
- Do not expand `.mm` surface area for new feature modules by default.
- Existing Objective-C++ files are maintained with SRP refactors and bug fixes only; avoid large new `.mm` subsystems.
- New macOS capabilities (new UI panels, new system integrations) should prefer Swift-first modules.
- Swift build mode must use latest stable language semantics by default (`MFX_SWIFT_LANGUAGE_MODE=auto`, currently resolves to Swift 6 on Swift 6 toolchains, otherwise Swift 5).
- Adopt strangler migration: keep old `.mm` paths stable, route new capability entrypoints to Swift incrementally.
- Prefer Swift <-> C++ interop path for new modules; keep Objective-C++ as thin bridge only where legacy AppKit glue is unavoidable.

### Windows build fallback (when needed)
- Prefer MSBuild absolute paths if `msbuild` is not directly available.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqmw/MFCMouseEffect](https://github.com/sqmw/MFCMouseEffect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
