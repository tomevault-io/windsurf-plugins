---
trigger: always_on
description: This file provides guidance to AI coding agents for contributing to
---

# AGENTS.md - Guidelines for AI Contributions to Zulip

This file provides guidance to AI coding agents for contributing to
the Zulip codebase. These guidelines are designed to produce
contributions that meet the same high standards we expect from human
contributors.

When doing any work with AI assistants, it is crucial to follow our
AI use policy and guidelines (the "AI use policy and guidelines"
section in `CONTRIBUTING.md`). In particular:

> **Do not submit an AI-generated PR you haven't personally
> understood and tested**, as this wastes maintainers' time. PRs that
> appear to violate this guideline will be closed without review.

> **Do not post AI-generated messages** in the [Zulip development
> community](https://zulip.com/development-community/) -- we want to
> read your own genuine expression of your thoughts.

Claude Code automatically uses `.claude/rules/` and `.claude/skills/`.
If you are a different agent, read the files in these folders to see
the rest of the agent instructions and when they're relevant. Rules
are relevant for files listed in `paths:` at the top of the rule
file. Skills explain when they're relevant in their `description`.

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

There is no category of "minor issue" that is acceptable to ship —
if something is broken in any state, size, theme, or language where
a user would encounter it, it must be fixed before merging. If a fix
would require a design decision, raise it as a question rather than
shipping the broken state. See `.claude/rules/ui-testing.md` for
what to test for UI changes (the file loads automatically when you
work on frontend files).

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

### 2. Propose an Approach

Before writing code, explain the plan:

- Explain your understanding of the problem and all relevant design decisions
- What changes are needed and why
- How the changes fit with existing patterns
- What could break and how to prevent regressions

### 3. Implement in Minimal, Coherent Commits

Structure changes as clean commits:

- Backend and API changes, with tests and API doc changes documented
  fully using our double-entry changelog system. Instructions for
  documentation can be found in `.claude/rules/api-changelog.md`
  (loaded automatically when you edit `zerver/openapi/zulip.yaml`).
- Frontend UI changes (with tests and user-facing documentation
  updates). Remember to plan to use your visual test skill
  (`.claude/skills/visual-test/SKILL.md`) to check your work whenever
  you change web app code (HTML, CSS, JS).

Each commit should be self-contained, highly readable and reviewable
using `git show --color-moved`, and pass lint/tests independently. If
extracting new files or moving code, always do that in a separate
commit from other changes.

### 4. Verify Before Finalizing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zulip/zulip](https://github.com/zulip/zulip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
