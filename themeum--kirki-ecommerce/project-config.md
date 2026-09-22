---
trigger: always_on
description: This file is based on `.cursor/rules/` but the PHP and React sections have
---

# Project Instructions for Claude

This file is based on `.cursor/rules/` but the PHP and React sections have
been re-derived from the actual codebase (not just copied from the `.mdc`
files), so they reflect real conventions rather than stale ones — e.g. the
frontend moved from `.jsx` to TypeScript, and the PHP `@since`/`final` rules
didn't match what the code actually does. Section 1 (behavioral guidelines)
is a direct mirror of `karpathy-guidelines.mdc`. If the codebase's conventions
change, re-derive rather than trusting `.cursor/rules/` at face value.

---

## 0. Testing / Verification

Do not use the Browser tool (or any dev-server preview) to test or verify
changes in this project. Skip the browser-based verification workflow
entirely — rely on typecheck (`npm run typecheck`), lint, and the test suite
(`npm test` in `resources/app/`) instead. If a change genuinely needs visual
confirmation, say so and let the user check it themselves rather than
opening a browser preview.

---

## 1. Behavioral Guidelines (always apply)

Source: `.cursor/rules/karpathy-guidelines.mdc`

Behavioral guidelines to reduce common LLM coding mistakes.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

### Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## 1a. Planning Workflow

When entering plan mode in this project, always use the **OpenSpec workflow**
instead of writing a freeform plan. Reach for the `openspec-*` / `opsx:*`
skills:

- `opsx:explore` — think through the problem before committing to a change
- `opsx:propose` — generate a full proposal (spec deltas, design, tasks)
- `opsx:apply` — implement tasks from an existing change
- `opsx:sync` — sync delta specs into main specs
- `opsx:archive` — finalize and archive a completed change

Before implementing tasks from an existing change always ask me to run the command `opsx:apply` manually
by myself instead of applying automatically.

Note: Whenever I start a new session make sure to follow the **OpenSpec workflow** by default.

---

## 2. PHP Coding Standards

Derived from analyzing the actual code in `app/` and `database/` (404 PHP files).
Applies to: `app/**/*.php`, `database/**/*.php`.

**Do not use `vendor/libraries/framework/src/` as a style reference, and do not
hand-edit it.** It is the `themeum/framework` package, relocated there from
`vendor/themeum/framework` and namespace-rewritten by the `composer scope`
script (`php-scoper` stages the prefixed output, then `bin/scope-framework.php`
swaps it in). Any edit is lost on the next `composer install`. Its conventions
belong to the upstream package, not this project.

Target PHP **7.4** (see `composer.json` `config.platform.php`). Follow PSR-4 file naming.

### WordPress.org Plugin Directory Requirements

This plugin targets wordpress.org submission. Apply the required-for-approval
subset of WordPress coding standards (escaping, sanitization/unslashing,
nonces, i18n, ABSPATH guards, WP-version compatibility, no global PHP state
mutation) to every PHP change, in every session — not just when a task is
explicitly about submission readiness. This is narrower than full

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themeum/kirki-ecommerce](https://github.com/themeum/kirki-ecommerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
