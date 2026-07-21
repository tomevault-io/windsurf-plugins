---
trigger: always_on
description: The defaults below apply to every change in this codebase.
---

# General

## Engineering Principles

The defaults below apply to every change in this codebase.

- **Type safety throughout.** Favor strict typing, exhaustive checks, and discriminated unions. `any` and `unknown` are escape hatches, not load-bearing tools — if you reach for one, justify it.
- **Clean architecture.** Keep concerns separated, dependencies pointing inward, business logic decoupled from frameworks and I/O. The renderer shouldn't know how the main process fetches; the main process shouldn't know which component renders.
- **Prefer mature libraries over bespoke code.** For validation, HTTP, ORM, auth, logging, and similar plumbing, reach for the production-proven, widely-adopted package before writing your own. If a mature library solves the problem, use it.
- **Push back when something looks wrong.** Don't defer to the user reflexively. If a proposal looks incorrect, incomplete, or inconsistent with the codebase, name the specific reason rather than agreeing. Agreement-by-default is a failure mode.
- **Verify before you act on uncertainty.** When you're not confident about a fact, an API surface, a library's behavior, or how a piece of this codebase works, check it — read the source, check the docs, run a quick test. Assumed knowledge is the most common source of bugs.
- **Reuse the domain glossary** in `CONTEXT.md` — the canonical naming reference for the architecture. Use those terms exactly when extending or refactoring; add new entries when extracting modules instead of letting parallel vocabularies sprout.
- Don’t fight errors! Whenever you encounter the same error 3 times, research the web (and github using gh api) and find 3-5 possible ways to fix it. Then choose the most efficient solution and implement it or stop to notify user that there is no clean fix at this point.

---

## Domain Glossary

Use the canonical glossary in [`CONTEXT.md`](./CONTEXT.md).

The glossary is the single source of terminology for this repo. Keep it implementation-free (no module names, file paths, or migration notes) and update it when vocabulary decisions change. This prevents ambiguous PR reviews and keeps architecture and UX aligned.

### What is an ADR?

An ADR (Architecture Decision Record) is a short, high-signal note for a design decision that is:

- hard to reverse,
- surprising without context,
- and chosen through a real trade-off.

Use `docs/adr/0001-slug.md` style files for those decisions. Keep them concise.

### ADR creation rules

- Create [`docs/adr/`](./docs/adr/) only when the first ADR is needed.
- Use sequential file names: `0001-...`, `0002-...`, and so on.
- Skip ADRs for reversible or obvious implementation choices.

---

## Working Conventions

**No backward compatibility** — Do not prioritize migration paths, backward compatibility shims, or preserving existing implementations. Existing code can be discarded or restructured without ceremony. Skip any "keep for compat" hedging. Delete old types, old components, old state fields without re-exporting or aliasing. Refactor aggressively. No deprecation notices or transitional layers.

**schemas.ts is SSOT for all enum types.** Pattern: `z.enum([...])` → `type Foo = z.infer<…>` → `const FOOS = fooSchema.options`. Don't redeclare as TS union literals. `QUEUE_STATUS` / `STATUS_KEY` live in `schemas.ts` (not `constants.ts`). `DEFAULTS` lives in `constants.ts`.

**TDD for non-trivial changes.** Write failing tests first, implement minimally, then refactor. Skip only for typo fixes and single-line edits.

**Translation workflow.** Use the `translate-arroxy-i18n` project skill for app locale changes, PO/POT sync, generated locale JSON, and i18n audits.

**Session scope.** Treat every agent session as scoped to the task context it was given. Other agents may be working in parallel on the same branch, so do not repair, reformat, revert, or restructure files outside your current scope just because you notice a problem there.

### Agent Skills

- **translate-arroxy-i18n** — manages app locale updates, gettext PO/POT sync, runtime locale JSON generation, and i18n audit commands. Entry point: `.agents/skills/translate-arroxy-i18n/SKILL.md`.

---

## Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State assumptions explicitly. Ask if uncertain.
- Multiple interpretations? Present them — don't pick silently.
- Simpler approach exists? Say so. Push back when warranted.
- Unclear? Stop. Name what's confusing. Ask.

## Goal-Driven Execution

**Define success criteria. Loop until verified.** Transform tasks into verifiable goals: "Add validation" → tests for invalid inputs, then make them pass. "Fix the bug" → reproduce-test, then make it pass. "Refactor X" → tests pass before and after. For multi-step tasks, state a brief plan with a `verify:` line per step.

For multi-step tasks, use this shape:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria like "make it work" require constant clarification.

**Simplify aggressively.** If you write 200 lines and it could be 50, rewrite it. Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

---

# Project-Scoped


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonio-orionus/Arroxy](https://github.com/antonio-orionus/Arroxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
