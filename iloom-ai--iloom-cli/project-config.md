---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

iloom is a TypeScript CLI tool that converts existing bash workflow scripts into a robust, testable system for managing isolated Git worktrees with Claude Code integration. The tool enables developers to work on multiple issues simultaneously without context confusion.

## DEVELOPMENT GUIDELINES
* DO NOT SWALLOW ERRORS
* Use Exception throwing, do not use "CommandResult" objects that return "success: true | false" - it either returns successfully or not at all.
* When catching exceptions and returning objects or throwing new exceptions, you must be very specific about the circumstances in which you are doing that. You must explicitly check for the expected error class, message (or substring) or code before returning an object or throwing a new error. Failure to do this effectively swallows the error.
* Use pnpm as your package manager. Don't use npm.
* **Avoid dynamic imports**: Use static imports at the top of files unless there's a genuine need for lazy loading (e.g., CLI commands that may not be invoked) or breaking circular dependencies. Dynamic imports add complexity, hurt performance, and make dependencies harder to trace. Before adding a dynamic import, check if the module is already imported elsewhere in the file or if a static import would work.
* **ALWAYS run `pnpm build` after completing major tasks** to ensure the TypeScript builds successfully and make the functionality available for testing. This catches compilation errors early and enables users to test new features immediately. Major tasks include: implementing new features, refactoring code, adding/modifying CLI commands, or making significant changes to core modules.
* **Identifier comparisons are case-sensitive risks.** `Map.get`/`Map.has`, `Set.has`, `Array.includes`, and `===` on issue identifiers, branch names, or URL components are case-sensitive in JS — `'WEB-2423' !== 'web-2423'`. Before assuming a comparison is safe, trace BOTH sides to where they were written. If they're populated by different code paths or stored in different fields, treat as risky. Either normalize at write time via `IssueTracker.normalizeIdentifier()`, or do a case-insensitive comparison defensively (handles legacy data already on disk). "Pure iteration" describes how data moves, not whether keys match across sources.

### Telemetry Requirements

**Purpose:** Telemetry helps us understand which features are actually used, where users hit errors, and how workflows perform in practice. This data drives decisions about what to improve, what to deprecate, and where to invest effort. Without it, we're guessing.

**What to track:** Add telemetry when adding new commands, features, or significant user-facing workflows. Specifically:
- **Command usage**: When a new CLI command or subcommand is added, track that it was invoked and whether it succeeded
- **Feature adoption**: When adding a new option, mode, or integration, track which variants users choose (e.g., tracker type, merge behavior, one-shot mode)
- **Workflow outcomes**: Track success/failure of multi-step workflows (loom lifecycle, swarm execution, epic planning) with duration and outcome counts
- **Error rates**: Track error types (NOT messages) at top-level catch boundaries so we can identify reliability issues
- **Lifecycle events**: Track install, upgrade, and session start to understand the active user base and version distribution

**You do NOT need to track:** Internal helper functions, intermediate steps within a workflow, or read-only operations (list, status, config display).

The telemetry system is in `src/lib/TelemetryService.ts` and event types are defined in `src/types/telemetry.ts`.

**How to add telemetry:**
- Import `TelemetryService` and call `TelemetryService.getInstance().track('event.name', { properties })` at the success point of the workflow
- All tracking calls must be wrapped in try/catch with `logger.debug()` — telemetry must NEVER break user workflows
- Tracking calls are fire-and-forget (non-blocking)
- If adding a new event type, define its interface in `src/types/telemetry.ts` and add it to the `TelemetryEventMap`

**CRITICAL — Anonymity and privacy rules for telemetry properties:**
- **NEVER** include repository names, URLs, or remote origins
- **NEVER** include branch names, issue titles, issue descriptions, or issue content
- **NEVER** include file paths, code content, or AI-generated analysis/plan content
- **NEVER** include GitHub/Linear/Jira usernames, emails, or any user identifiers
- **NEVER** include error messages (they can contain file paths or PII) — only use `error.constructor.name` for error types
- **DO** include: counts (child_count, duration_minutes), enums (tracker type, merge behavior, source type), booleans (success/failure, feature flags), and the CLI version
- When in doubt, ask: "Could this property identify a specific person, project, or repository?" If yes, do not include it.

**Existing patterns to follow:**
- `src/commands/start.ts` — tracking `loom.created` after successful start
- `src/commands/finish.ts` — tracking `loom.finished` with duration calculation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iloom-ai/iloom-cli](https://github.com/iloom-ai/iloom-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
