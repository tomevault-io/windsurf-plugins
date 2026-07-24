---
trigger: always_on
description: generates an empty `api_docs/unmerged.d/ZF-XXXXXX.md` file (where
---

# CLAUDE.md - Guidelines for AI Contributions to Zulip

This file provides guidance to Claude (and other AI coding assistants) for
contributing to the Zulip codebase. These guidelines are designed to produce
contributions that meet the same high standards we expect from human
contributors.

## Philosophy

Zulip is a team chat application used by thousands of organizations,
built to last for many years. It is developed by a vibrant open-source
community, with maintainers who have consistently emphasized **high
standards for codebase readability, code review, commit discipline,
debuggability, automated testing, tooling, documentation, and all the
other subtle details that together determine whether software is easy
to understand, operate, and modify**.

Zulip's engineering strategy is to **"move quickly without breaking
things"**. This is possible because the project has invested years in
testing, tooling, code structure, documentation, and development
practices that catch bugs systematically rather than relying on
individual vigilance. Maintainers spend most of their review time on
product decisions and code structure/readability, not on chasing
correctness issues — because the process is designed to prevent them.

This means Zulip's coding philosophy is to **focus relentlessly on
making the codebase easy to understand and difficult to make dangerous
mistakes**. This applies equally to AI-generated contributions. Every
change should make the codebase more maintainable and easier to read.

### No detail is too small

Zulip holds itself to a high bar for polish because users depend on
this software daily, and because the project is built to last for
decades. There is no category of "minor issue" that is acceptable
to ship — if something is broken in any context where a user would
encounter it, it must be fixed before merging. The project's
extensive investment in testing, tooling, and review processes exists
precisely so that these issues get caught and fixed, not so that they
can be classified as low-priority and deferred.

This philosophy extends to every aspect of the product:

- **Visual precision matters.** Alignment, spacing, colors, and font
  sizes must be consistent with similar existing UI. When making CSS
  changes, you must demonstrate with pixel-precise before/after
  comparisons that there are no unintended side effects.
- **Every state matters.** UI must look correct in all its states:
  hover, active, disabled, focused, selected, empty, overflowing.
  Changes that could plausibly affect colors, contrast, or
  theme-dependent imagery must work in both light and dark themes;
  changes whose effect can't reasonably vary with theme (pure
  geometry/typography — `font-size`, `line-height`, `margin`,
  `padding`, `display`, `font-weight`, etc.) only need a single
  theme verified.
- **Every window size matters.** UI must look good from wide desktop
  (1920px) down to narrow phone screens (480px).
- **Every language matters.** Translated strings can be 1.5x longer
  than English or half as short. UI must handle both extremes without
  breaking layout. Think about right-to-left languages too.
- **Every interaction path matters.** Keyboard navigation, screen
  readers, permission levels, feature interactions (banners
  overlapping, resolved topics, muted messages), and edge cases in
  data (empty lists, very long names, single items vs. many) must all
  be considered.

The right attitude is: "What could go wrong, and how do I verify that
it doesn't?" not "It looks fine to me." **What isn't tested probably
doesn't work** — this applies to visual changes just as much as to
backend logic.

### Understand before coding

Before writing any code, you must understand:

1. What the existing code does and why, including the relevant help center or
   developer-facing documentation.
2. What problem you're solving, in its full scope.
3. Why your approach is the right solution, and available alternatives.
4. How you will verify that your work is correct, and avoid regressions
   that are plausible for the type of work you're doing.

The answer to "Why is X an improvement?" should never be "I'm not sure."

## Workflow

Follow this workflow for every task: **understand → propose → implement → verify**.

### 1. Understand Before Coding

Before making any changes:

```bash
# Read relevant documentation
cat docs/*/<relevant-area>.md
cat starlight_help/src/content/docs/<topic>.md
cat api_docs/<topic>.md and read the relevant part of zerver/openapi/zulip.yaml

# Look at existing code patterns
git grep "similar_function_name"
git log --oneline -20 -- path/to/file.py

# Check for related issues on GitHub
```

Always show existing similar code and explain how it works before proposing
changes.

### 2. Propose an Approach

Before writing code, explain the plan:

- Explain your understanding of the problem and all relevant design decisions
- What changes are needed and why
- How the changes fit with existing patterns
- What could break and how to prevent regressions

### 3. Implement in Minimal, Coherent Commits

Structure changes as clean commits:

- Backend and API changes (with tests and API doc changes documented
  fully using our double-entry changelog system). When starting an API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zulip/zulip](https://github.com/zulip/zulip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
