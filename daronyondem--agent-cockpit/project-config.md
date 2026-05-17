---
trigger: always_on
description: This file is the canonical project guidance for coding agents working in this repository. Keep it vendor-neutral when possible; use tool-specific files such as `CLAUDE.md` only as compatibility shims or for genuinely tool-specific notes.
---

# Agent Instructions

This file is the canonical project guidance for coding agents working in this repository. Keep it vendor-neutral when possible; use tool-specific files such as `CLAUDE.md` only as compatibility shims or for genuinely tool-specific notes.

Project memory that should persist across agents is mirrored in
[`docs/agent-project-memory.md`](docs/agent-project-memory.md). Read it when
work touches installer/updater/release behavior, CLI profiles, mobile PWA,
Context Map, Memory, KB, or cross-agent workflow. When durable memory records a
recurring project rule, update that document and this file if the rule changes
agent workflow.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

# Maintainability Boundaries

Follow ADR-0051 for shared contracts, logging, and ownership boundaries.

- Keep public facades stable. `ChatService` and `ContextMapService` should remain orchestration/facade entrypoints; new persistence, parsing, normalization, scoring, source-planning, or policy logic should live in focused modules under the owning domain directory.
- For ChatService work, prefer focused modules under `src/services/chat/` for artifacts, usage ledgers, workspace feature settings, message queues, workspace instructions, and similar private stores. Preserve existing storage formats and migrations unless the task explicitly changes them.
- For Context Map work, keep `src/services/contextMap/service.ts` as the coordinator. Put source discovery/planning in `sourcePlanning.ts`, candidate primitives in `candidatePrimitives.ts`, safe auto-apply policy in `autoApply.ts`, JSON extraction/repair in `jsonRepair.ts`, and run/synthesis metadata in `pipelineMetadata.ts`.
- Put shared request/response shapes and runtime validators in `src/contracts/` whenever routes, clients, tests, or specs need the same boundary. Contract files imported by web or mobile must stay browser-safe and must not import server-only modules.
- Web and mobile clients should import shared types only from browser-safe contracts. Do not reach into backend services, route modules, filesystem helpers, or server-only types from frontend code.
- Keep large UI entrypoints as composition layers. Move pure projection, parsing, state-provider, viewport, and formatting behavior into focused modules or hooks with direct tests.
- Prefer behavior-oriented frontend tests for parsing, projection, state updates, and user-visible outcomes. Use source-string/static tests only for route registration, build guards, or import-boundary checks where runtime coverage is impractical.
- Use `src/utils/logger.ts` for backend logging in touched code. Do not add new backend `console.*` calls unless the file is an intentional CLI/test/build script or already documented as an allowed exception.
- When adding or changing an ownership boundary, update the relevant spec docs and add focused tests for the new module instead of only testing through the largest facade.
- Before opening a PR, run `npm run maintainability:check` and `npm run spec:drift` in addition to the existing required verification commands.

# Claude Code Interactive Compatibility


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daronyondem/agent-cockpit](https://github.com/daronyondem/agent-cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
