---
trigger: always_on
description: This file contains preferences and conventions for Claude when working on this codebase.
---

# Claude Development Notes

This file contains preferences and conventions for Claude when working on this codebase.

## Project Conventions

- "Foldkit" is always capitalized in prose — in READMEs, docs, commit messages, comments, and conversation. The only exception is the npm package name (`foldkit`) and import paths (`from 'foldkit/html'`).
- In prose (docs, comments, conversation), capitalize Foldkit architecture concepts that correspond to actual types or named patterns: Model, Message, Command, Subscription, Task, Submodel, OutMessage. Keep lowercase for concepts that are just functions with no corresponding type or pattern: view, update, init.
- This is a Foldkit project — a framework built on Effect-TS. Always use Schema types (not plain TypeScript types), full names like `Message` (not `Msg`), and `withReturnType` (not `as const` or type casting). Follow the Submodels and OutMessage patterns used throughout the codebase.
- Foldkit is tightly coupled to the Effect ecosystem. Do not suggest solutions outside of Effect-TS. The project already has a `create-foldkit-app` scaffolding tool — check existing features before suggesting new ones.
- Push back on any suggested direction that violates Elm Architecture principles — unidirectional data flow, Messages as facts (not commands), Model as single source of truth, and side effects confined to Commands. If a user or prompt suggests a pattern that breaks these conventions (e.g. mutating state directly, imperative event handlers, two-way bindings), flag the issue and propose the idiomatic Foldkit approach instead.

## Code Quality Standards

Before writing code, read the exemplar files to internalize the level of care expected:

Library internals (when working in `packages/foldkit/src/`):

- `packages/foldkit/src/runtime/runtime.ts` — orchestration, state management, error recovery
- `packages/foldkit/src/parser.ts` — bidirectional combinators, type-safe composition

Application architecture (when working in `packages/website/`, examples, or apps built with Foldkit):

- `examples/typing-game/client/src/` — Submodels, OutMessage, update/Message patterns, view decomposition, Commands

Match the quality and thoughtfulness of these files. The principles below apply broadly, but calibrate to the right context — library design when building Foldkit internals, application architecture when building with Foldkit:

- Every name should eliminate ambiguity. Prefix Option-typed values with `maybe` (e.g. `maybeCurrentVNode`, `maybeSession`). Name functions by their precise effect (e.g. `enqueueMessage` not `addMessage`). A reader should never need to check a type signature to understand what a name refers to.
- Each function should operate at a single abstraction level. Orchestrators delegate to focused helpers — they don't mix coordination with implementation. If a function reads like it's doing two things, extract one.
- Encode state in discriminated unions, not booleans or nullable fields. Use `Idle | Loading | Error | Ok` instead of `isLoading: boolean`. Use `EnterUsername | SelectAction | EnterRoomId` instead of `step: number`. Make impossible states unrepresentable.
- Name Messages as verb-first, past-tense events describing what happened (`SubmittedUsernameForm`, `CreatedRoom`, `PressedKey`), not imperative commands. The verb prefix acts as a category marker: `Clicked*` for button presses, `Updated*` for input changes, `Succeeded*`/`Failed*` for Command results that can meaningfully fail (e.g. `SucceededFetchWeather`, `FailedFetchWeather`), `Completed*` for fire-and-forget Command acknowledgments where the result is uninteresting and the update function is a no-op (e.g. `CompletedLockScroll`, `CompletedShowDialog`, `CompletedNavigateInternal`), `Got*` exclusively for receiving child module results via the Submodel pattern (e.g. `GotTransitionMessage`, `GotProductsMessage`). The update function decides what to do — Messages are facts.
- Never use `NoOp` as a Message. Every Message must carry meaning about what happened. Fire-and-forget Commands use `Completed*` Messages with verb-first naming that mirrors the Command name: Command `LockScroll` → Message `CompletedLockScroll`, Command `ShowDialog` → Message `CompletedShowDialog`, Command `FocusInput` → Message `CompletedFocusInput`. View-dispatched no-ops use descriptive facts: `IgnoredMouseClick`, `SuppressedSpaceScroll`.
- Use `Option` for values that flow through chains or pattern matching — `Option.match`, `Option.map`, `Option.flatMap`. Use `Option.fromNullishOr` at boundaries where the value will be matched or chained, not as a verbose synonym for `!== undefined`. Simple presence checks are fine when you're not chaining — don't wrap in Option just to immediately check `isSome`. Prefer `Option.match` over `Option.map` + `Option.getOrElse` — if you're unwrapping at the end, just match. Use `OptionExt.when(condition, value)` instead of `condition ? Option.some(value) : Option.none()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foldkit/foldkit](https://github.com/foldkit/foldkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
