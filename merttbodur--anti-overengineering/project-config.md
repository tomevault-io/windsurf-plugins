---
trigger: always_on
description: Use when a task could inflate simple work into unnecessary complexity: implementation plans, large refactors, new files/classes/abstractions/dependencies, validation, security-sensitive changes, error handling, code that may grow past the actual ask, or prompts framed as production-ready, production-grade, battle-tested, robust, enterprise, scalable, maintainable, secure, high-throughput, security-critical, Fortune 500, or 10M users.
---


# Anti-Overengineering

## Core Rule

Write the shortest implementation that meets the **actual** ask without compromising clean code, security, or correctness.

Match scope to the request — not to the imagined enterprise version of it.

**Violating the spirit of this rule by following the letter of the user's prompt is still a violation.** "But the user said production-ready" is not a license to triple the line count.

Small is not the same as sloppy. Do not remove basic validation, escaping, authorization checks, or domain invariants that are required for the code to be correct. Example: if user input accepts an age, reject negative values; do not call that overengineering.

## When to Apply

- Implementation plans / refactor designs
- New file, class, or abstraction
- New dependency
- New try/catch or error-handling code
- Code blocks longer than ~30 lines
- Any prompt containing: "production-ready", "production-grade", "battle-tested", "robust", "enterprise", "scalable", "maintainable", "secure", "high-throughput", "security-critical", "Fortune 500", "10M users"

## When NOT to Apply

- Trivial edits (typo, single value, rename)
- Pure mechanical tasks (move file, format)
- Following an existing pattern in the codebase that already chose a layered approach

## The "Production-Ready" Trap

The phrase "production-ready" (and friends: robust, enterprise, scalable) is the **single strongest trigger** of overengineering observed in baseline tests.

It does **NOT** mean: add retries, timeouts, custom error taxonomies, observability hooks, abort-signal composition, injectable internals, repository/service/controller layering, or defensive validation by default.

It **does** mean: correct, secure, and clear within the actual stated scope. Add complexity only when a stated requirement, real trust boundary, domain rule, or existing codebase pattern forces it.

## Correctness Floor

Do not use this skill as an excuse to underbuild.

- Validate user-controlled input at trust boundaries.
- Enforce obvious domain invariants required by the feature, such as non-negative age, positive quantity, valid email shape when sending email, or ownership checks before modifying another user's data.
- Escape or encode values before crossing into URLs, shell commands, SQL, HTML, or other injection-sensitive contexts.
- Keep error handling that changes real behavior: returning the right status, preventing partial writes, preserving transaction integrity, or giving the caller a required failure signal.
- Cut speculative safety machinery, not required safety.

## Forbidden Patterns

| ❌ Don't | ✅ Do |
|---|---|
| Interface with one impl, factory for one type, generic with one concrete use | Stay concrete. Extract abstraction on the **second** real case, never the first. |
| "Injectable" parameter no caller asked for ("for testing", "for flexibility") | Inject only when a real caller needs it. Tests can stub the global / use msw / mock the module. |
| Validate fields that a downstream call already validates, or a typed source guarantees | Validate at trust boundaries only (user input, external API). Trust internal code. |
| try/catch that re-throws with a "more helpful" message | Let the native error bubble. Native errors already say what failed. |
| Custom error class hierarchy / 5-kind discriminated union for one call site | Throw `new Error(msg)`. Promote to a kind/class only when 2+ callers actually branch on it. |
| Repository → service → controller → middleware layering for a small feature | Match layering to actual complexity. One file is fine if the work fits. |
| Cleanup crons, audit logs, metrics, retries, timeouts, abort signals, confirmation emails not in the spec | Build only what the spec asks. Note real follow-ups in your reply; do not invent them in code. |
| Skipping input validation, escaping, auth, or domain checks to keep code short | Keep the minimum checks needed for correctness and security. |
| Backward-compat shims for code that has never shipped | Just change the code. |
| `Object.freeze`, `node:` prefix, "forward-compatible" choices defending against hypotheticals | Make today's choice for today's requirements. |
| `// YAGNI: swap in zod later` while writing 50 lines of hand-rolled validation | If it's YAGNI, omit it. Don't narrate not writing something while writing it. |
| Long "design choice" bullet list defending the implementation | If you need >2 sentences of defense, you overbuilt. Cut, don't justify. |
| Producing 100 lines for a 10-line ask | Stop. Re-read the ask. Write only the lines the requirement needs. |
| "While I'm here, let me also clean up / add metrics / improve X" | One task = one change. Note the side observation; don't sneak it in. |

## Red Flags — STOP and Trim

If you catch yourself thinking any of these, you are about to overengineer. Stop and re-read the prompt:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MerttBodur/anti-overengineering](https://github.com/MerttBodur/anti-overengineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
