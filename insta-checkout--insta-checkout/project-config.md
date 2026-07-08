---
trigger: always_on
description: Follow these rules unless the user explicitly instructs otherwise.
---

# Insta Checkout — Agent Instructions

Follow these rules unless the user explicitly instructs otherwise.

## Critical Rules

- **Do NOT commit** any work until the user explicitly asks you to.
- **Do NOT deploy** any work until the user explicitly asks you to.
- **Do NOT push** to remote until the user explicitly asks you to.
- **Do NOT merge main** into the current branch unless the user explicitly asks you to.
- **Do NOT access any production data store** without explicit, in-the-moment user approval. This covers — at minimum — the **production MongoDB cluster**, the **production Firebase project** (Auth, Storage, Firestore, Realtime Database), and any other prod-side data store the project adds later. Read, write, delete, query, admin-SDK calls, CLI commands, scripts, and one-off connections all count as "access". Working against dev/staging/PR-env data stores is fine and does not need approval. When in doubt about whether a connection string, project ID, bucket name, or service account belongs to prod, **stop and ask** — never guess. If a task seems to require touching prod, propose the action and wait for an explicit "yes, go ahead with prod" before proceeding.

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions).
- If something goes sideways, STOP and re-plan immediately — don't keep pushing.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.
- **Exception**: Skip plan mode during Night Shift — execute tasks directly without entering plan mode.

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, throw more compute at it via subagents.
- One task per subagent for focused execution.

### 3. Self-Improvement Loop
- After ANY correction from the user: update memory files with the pattern.
- Write rules for yourself that prevent the same mistake.
- Review lessons at session start for relevant project.

### 4. Verification Before Done
- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution."
- Skip this for simple, obvious fixes — don't over-engineer.
- Challenge your own work before presenting it.

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- **Follow Existing Patterns**: Match the codebase's existing structures and conventions.

## Brand Voice (MANDATORY)

All user-facing text in the **landing**, **checkout**, and **admin** apps — in **both Arabic and English** — MUST follow the Brand Voice Guide. It is the single source of truth for how Insta Checkout communicates:

➡️ https://app.notion.com/p/karimtamer/Brand-Voice-Guide-31cc92f98d9c81acbb1ae8cfd0e217b4

**Scope:** any copy a user reads — UI labels, buttons, headings, body copy, onboarding, empty/error/success states, tooltips, emails, and the i18n strings in `packages/i18n/messages/*.json` (both `en.json` and `ar.json`). This applies whenever you add or change text, not just at commit time.

**Before writing or changing any user-facing string, fetch the page** (Notion connector) so you are working from the current guidance. If Notion is unreachable (e.g. headless / night-shift runs), fall back to the distilled rules below — but the live page wins on any conflict.

### Voice in one line
Write like a **trusted local expert**: trustworthy, clear and direct, locally aware (built for Egypt / InstaPay), warm and human. The voice is fixed; only the tone shifts by channel — product UI is clear and neutral, onboarding is warm and encouraging, error messages are empathetic and never blame the user.

### Always
- Plain, short sentences. Every sentence must earn its place.
- Sentence case for headings ("Create a payment link").
- Oxford comma; `%` symbol; spell out one–nine, numerals for 10+.
- Contractions in conversational copy; at most one exclamation mark per piece, only when earned.
- Bold for emphasis — never ALL CAPS.

### Never
- Hype words: "revolutionary", "game-changing", "disrupting", "seamless", "easy", or a loose "empower".
- "solution" when you mean product/tool; "user" / "merchant" / "vendor" when you mean **seller**.

### Preferred terminology
payment link (not paylink) · seller (not user/merchant/vendor) · dashboard · sign up / log in (verbs) · InstaPay · Insta Checkout · funds (not money/cash in UI).

### Arabic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Insta-Checkout/insta-checkout](https://github.com/Insta-Checkout/insta-checkout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
