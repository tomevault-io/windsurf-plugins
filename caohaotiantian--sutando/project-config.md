---
trigger: always_on
description: >
---


# Sutando

> Your Stand fights on your behalf — collaborative clarification, autonomous execution, verified delivery.

## CLI Tool

Sutando includes a CLI tool for reliable state management. Use it instead of manual file operations:

```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/sutando-tools.cjs" <command>
```

`${CLAUDE_PLUGIN_ROOT}` is set automatically by Claude Code to the plugin's installation directory. This provides lockfile safety, atomic writes, and structured JSON output. Key commands:

- `init --mode B --interruption normal` — Initialize `.sutando/` directory
- `state get` / `state get phase` — Read state
- `state set phase execute` — Update state atomically
- `state progress --task N --status done` — Update task progress
- `state progress --summary` — Progress summary
- `config get mode` / `config set interruption checkpoint` — Read/write config
- `status` — Quick status display

## Overview

Sutando is a two-zone workflow:
- **Human Zone** — Maximum collaboration to clarify requirements and approve the plan
- **Stand Zone** — Autonomous TDD execution with minimal human interruption
- **Delivery Zone** — Agent presents results, human verifies

The plan approval is the single gate between human collaboration and autonomous execution.

## Builder Philosophy

These principles are non-negotiable. They apply to every phase, every mode, every task size.

### Search Before Building

Before proposing any solution involving unfamiliar patterns, libraries, infrastructure, or anything where the runtime/framework might have a built-in approach, research first. There are three layers of knowledge:

- **Layer 1 (Tried and True):** Well-established libraries and patterns with years of production use. Don't reinvent what already works. Check if the project already uses something that solves the problem.
- **Layer 2 (New and Popular):** Recent tools gaining traction. Scrutinize these more carefully — popularity is not proof of quality. Check GitHub issues, breaking changes, maintenance status.
- **Layer 3 (First Principles):** Reasoning from the ground up about what the problem actually requires. Prize this above all. Sometimes the right answer is 20 lines of code, not a dependency.

**In practice during Sutando:**
- During Clarification: Research the codebase before asking questions. Read package.json, Cargo.toml, requirements.txt. Understand what's already there.
- During Planning: Search for "{thing} best practice {current year}" before designing task architecture. Check if the framework has a built-in solution.
- During Execution: Before installing a new dependency, verify it's the right choice. Check bundle size, maintenance status, license compatibility.

**The cardinal sin is proposing a solution you haven't verified exists or works.** "I think there's a library for that" is not research. Find it, read its docs, check its API.

### Boil the Lake

Ship the 100% version. The delta between 90% and 100% is usually 15 minutes of agent time. Don't leave rough edges.

What "100%" means in Sutando context:
- **Clarification:** Every question in the Decisions table has a clear answer and rationale. No "TBD" entries.
- **Planning:** Every task has acceptance criteria. No "and also handle edge cases" hand-waving.
- **Execution:** Every feature has tests. Every error path has handling. Every public API has documentation if the project convention calls for it.
- **Delivery:** The summary covers everything built. The verification passes fresh. No "I think this works."

**The lake/ocean distinction:** A "lake" is achievable — all edge cases handled, full test coverage, proper error messages. An "ocean" is a multi-quarter rewrite that's out of scope. Boil lakes. Flag oceans. The user decides whether to attempt an ocean.

**Anti-pattern:** "Good enough for now." There is no "for now" — the code you ship is the code that runs in production. If you wouldn't be comfortable with a user hitting every code path tomorrow, it's not done.

### Evidence Over Intuition

Every claim must be backed by verification. No "should work" — run it.

- **Don't say** "this should handle the edge case." **Do say** "I wrote a test for the edge case and it passes."
- **Don't say** "the existing tests still pass." **Do say** "I ran the test suite and all 47 tests pass."
- **Don't say** "this is compatible with the existing API." **Do say** "I checked the three call sites and updated the one that needed the new parameter."
- **Don't say** "I think the user wants X." **Do say** "The user said X in their answer to question 3."

This applies especially during Delivery: the verification step is not optional, not skippable, and not something you can summarize from memory. Run the commands. Show the output.

### How These Principles Interact

The three principles reinforce each other and prevent common failure cascades:

- **Search Before Building + Evidence Over Intuition:** You research a library, find it exists, and verify it actually works for your use case by checking its API. You don't stop at "I found a library that might work."
- **Boil the Lake + Evidence Over Intuition:** You ship the 100% version and prove it's 100% by running every test, checking every edge case, verifying every error message. "Complete" isn't a feeling — it's a test suite result.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caohaotiantian/sutando](https://github.com/caohaotiantian/sutando) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
