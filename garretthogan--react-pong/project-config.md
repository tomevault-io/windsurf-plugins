---
trigger: always_on
description: Scope: UI, APIs, async workflows, error handling, state management
---


# HCE Lint Ruleset (Human-Centered Engineering)
Version: 1.0
Scope: UI, APIs, async workflows, error handling, state management
Default severity: error

> Intent: Enforce discoverability, feedback, state transparency, error recovery, and predictable behavior.
> Apply to all new code and refactors. Prefer explicitness over cleverness.

---

## Severity Levels
- **error**: must fix before merge
- **warn**: fix before release; acceptable short-term with justification
- **info**: guidance; fix when touching related code

---

# Category: Async & Feedback

## hce/require-async-state
**severity:** error  
**rule:** Any async operation that affects user-facing flow must expose `idle | loading | success | error` states.  
**detect:** Functions returning Promise used in UI handlers without state tracking.  
**fix:** Introduce explicit state (state machine, reducer, query lib states) and render feedback.

## hce/no-silent-catch
**severity:** error  
**rule:** Catch blocks must not be empty or only log to console for user-facing flows.  
**detect:** `catch {}` or `catch (e) { console.* }` in UI pathways.  
**fix:** Surface actionable error + allow retry.

## hce/loading-over-150ms
**severity:** warn  
**rule:** Operations expected to exceed 150ms must show a loading indicator or skeleton.  
**detect:** Network calls without UI pending state.  
**fix:** Add spinner, skeleton, disabled button, progress.

## hce/prevent-double-submit
**severity:** error  
**rule:** Prevent duplicate submissions/clicks during pending async operations.  
**detect:** Form submit handlers without disabled state or idempotency.  
**fix:** Disable controls, debounce, or enforce idempotency keys.

## hce/optimistic-only-if-reversible
**severity:** error  
**rule:** Optimistic UI updates are allowed only if the action is reversible/rollbackable.  
**detect:** Immediate UI state mutation before server confirmation without rollback.  
**fix:** Add rollback path or switch to pessimistic update.

---

# Category: Errors & Recovery

## hce/user-errors-actionable
**severity:** error  
**rule:** User-facing errors must include **what**, **why**, and **how to fix**.  
**detect:** Messages like “Something went wrong”, “Failed”, or raw error strings.  
**fix:** Replace with structured message; include retry/support action.

## hce/no-stacktraces-to-users
**severity:** error  
**rule:** Never display stack traces, raw exceptions, or internal codes to end users.  
**detect:** Rendering error objects directly.  
**fix:** Map to user-safe errors; log details separately.

## hce/require-recovery-path
**severity:** error  
**rule:** Any error state must provide a recovery action (retry, undo, back, reset).  
**detect:** Error UI with no CTA.  
**fix:** Add retry button, “Try again,” restore defaults, etc.

## hce/destructive-needs-confirm-or-undo
**severity:** error  
**rule:** Destructive actions require confirmation OR undo.  
**detect:** Delete/Remove/Reset actions executed immediately.  
**fix:** Confirm modal, undo toast, soft delete, version history.

## hce/no-blame-copy
**severity:** warn  
**rule:** Error copy must not blame the user (“You did X wrong”).  
**detect:** Second-person accusatory text.  
**fix:** Use neutral system-language: “This couldn’t be completed because…”

---

# Category: State Transparency & Predictability

## hce/no-hidden-global-mutation
**severity:** error  
**rule:** Avoid hidden mutable global state that impacts user-visible behavior.  
**detect:** Module-scoped mutable objects used across flows; implicit singletons.  
**fix:** Explicit state container, dependency injection, or scoped stores.

## hce/state-change-visible
**severity:** error  
**rule:** User-visible state transitions must be visible in UI (saved/syncing/connected).  
**detect:** Background processes that alter data without indicators.  
**fix:** Add status chip, banner, inline state label.

## hce/no-surprise-automation
**severity:** warn  
**rule:** Automation that changes user data must be communicated and reversible where possible.  
**detect:** Auto-format, auto-delete, auto-move without prompt/indicator.  
**fix:** Add notice + undo; log in activity feed.

## hce/require-explicit-defaults
**severity:** error  
**rule:** Defaults must be explicit, safe, minimal, and reversible.  
**detect:** Implicit defaults buried in backend or magic constants.  
**fix:** Centralize defaults; document in code; avoid destructive defaults.

---

# Category: Discoverability & UI Signifiers

## hce/no-icon-only-without-label
**severity:** error  
**rule:** Icon-only controls must have accessible label + tooltip.  
**detect:** `<button><Icon/></button>` with no aria-label/title.  
**fix:** Add `aria-label`, `title`, visible label when space permits.

## hce/interactive-affordance-visible
**severity:** error  
**rule:** Primary interactables must look interactive (cursor, hover/focus states, boundaries).  
**detect:** Click handlers attached to non-interactive elements without styles.  
**fix:** Use semantic elements (`button`, `a`), add focus styles.

## hce/error-near-source
**severity:** error  
**rule:** Validation and input errors must render near the offending field.  
**detect:** Global error banners for per-field issues.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/garretthogan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
