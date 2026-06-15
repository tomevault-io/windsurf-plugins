---
trigger: always_on
description: Instructions for GitHub Copilot when generating code, reviewing PRs, and suggesting changes in this repository.
---

# Copilot Instructions

Instructions for GitHub Copilot when generating code, reviewing PRs, and suggesting changes in this repository.

## Project Overview

Agent Orchestrator (AO) is a TypeScript monorepo that manages fleets of parallel AI coding agents. Each agent gets its own git worktree, branch, and PR. The system handles CI feedback routing, review comment handling, and session lifecycle.

**Stack:** TypeScript (strict), pnpm monorepo, Next.js 15 + React 19, Tailwind CSS v4, Vitest, ESLint flat config.

**Architecture:** 8 plugin slots (Runtime, Agent, Workspace, Tracker, SCM, Notifier, Terminal, Lifecycle). All interfaces are defined in `packages/core/src/types.ts`. There is no database; the system uses flat files and memory.

Full conventions: `CLAUDE.md`. Plugin development: `docs/DEVELOPMENT.md`. Design system: `DESIGN.md`.

---

## Code Generation Rules

### Think Before Generating

- If a task is ambiguous, suggest the two most likely interpretations and ask which one applies. Do not choose silently.
- If there is a simpler approach than the one requested, say so. Push back when warranted.
- State assumptions explicitly when generating non-trivial code.

### Simplicity First

- No speculative features. No abstractions for single-use code. No "flexibility" that was not requested.
- Plugin slots are the extension point. If the user asks for configurability, consider whether a new plugin slot is the right answer instead.
- If you are generating 200 lines and it could be 50, rewrite it.
- Do not add error handling for impossible scenarios.

### Match Existing Patterns

- Before generating new code in an existing file, read how similar features are already implemented in that same file. Match the pattern.
- Do not introduce new patterns when established ones already exist. Search the codebase first.
- Match existing naming conventions, import styles, and file organization.
- Use `@aoagents/ao-core` for cross-package imports.
- Use the `workspace:*` protocol in `package.json`.

### TypeScript Strict Mode

- No `any` types unless they are in test files, where `any` and `console.log` are allowed.
- Use `import type { Foo }` for type-only imports.
- Prefix unused variables with `_`.
- Do not use `eval`, `new Function`, or `require()`; use ES module imports.

### Web / UI Specific

- Use Tailwind utility classes only. Do not use inline `style=` attributes.
- Use CSS custom properties via `var(--color-*)` from the `globals.css` `@theme` block. Never hardcode hex colors.
- Do not use external UI component libraries such as Radix, shadcn, or Headless UI.
- Preserve the dark theme at all times.
- Border radius must be `0px` everywhere except status dots and avatar circles. Hard edges are part of the visual identity.
- Mark client components with `"use client"`. Use server components for pages.
- SSE updates run at a 5-second interval via the `useSessionEvents` hook. Do not change this interval.
- Keep component files under 400 lines.

---

## PR Review Instructions

### What to Focus On

These are the areas where Copilot review adds the most value: issues CI cannot catch.

**1. Design over implementation.** A perfectly coded bad design is worse than a messy good one. Question:
- Side-channel communication, such as hidden flags or dynamic attribute setting
- Boolean parameters that switch between fundamentally different behaviors and should be separate code paths
- New internal contracts between components without interface documentation
- Missing migration paths for behavioral changes

**2. Pattern consistency.** If a file uses one pattern and the PR introduces a different one, flag it. Common violations:
- Using class attributes in one place and instance properties in another for the same concept
- Mixing callback styles when the file uses one style consistently
- Introducing a new error-handling pattern when the file uses `throw new Error("msg", { cause: err })`

**3. State machine safety.** Changes to `SessionStatus`, `ActivityState`, or lifecycle transitions require extra scrutiny:
- Verify that no invalid state transitions are introduced
- Check that `isTerminalSession()` and `TERMINAL_STATUSES` are updated if new statuses are added
- Flag any change that could cause a session to be incorrectly marked `killed` or `exited`

**4. Plugin interface stability.** Any change to interfaces in `types.ts` is potentially breaking:
- New required methods on plugin interfaces break all existing plugins
- Changed method signatures break all existing plugins
- New optional methods are acceptable
- Flag any non-optional interface change as "breaking — requires updating all N plugins implementing this slot"

**5. Backward compatibility.** Flag changes to:
- CLI flags or arguments in `packages/cli/`
- Config schema, including `agent-orchestrator.yaml` structure and Zod validation in `packages/core/src/config.ts`
- Exported types from `packages/core/src/index.ts`, which are a stable public API and should not break
- Default config values or behavior

**6. Plugin isolation.** Plugins must never import each other directly. They communicate through:
- The `Session` object
- The `LifecycleManager` event system
- Core utilities exported from `@aoagents/ao-core`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentWrapper/agent-orchestrator](https://github.com/AgentWrapper/agent-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
