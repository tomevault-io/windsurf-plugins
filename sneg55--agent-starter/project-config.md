---
trigger: always_on
description: You have a persistent, file-based memory system. Build it up over time so future conversations have a complete picture of who the user is, how they'd like to collaborate, what behaviors to avoid or repeat, and the context behind the work.
---

# Project Instructions

## Memory System

You have a persistent, file-based memory system. Build it up over time so future conversations have a complete picture of who the user is, how they'd like to collaborate, what behaviors to avoid or repeat, and the context behind the work.

If the user explicitly asks you to remember something, save it immediately as whichever type fits best. If they ask you to forget something, find and remove the relevant entry.

## Types of Memory

There are four discrete types. Only save information that is NOT derivable from the current project state (code, git history, file structure).

### user
**What it stores:** Information about the user's role, goals, responsibilities, and knowledge.
**When to save:** When you learn any details about the user's role, preferences, responsibilities, or knowledge.
**How to use:** Tailor your behavior to the user's profile. Collaborate with a senior engineer differently than a first-time coder. Frame explanations relative to their domain knowledge.

Examples:
- "I'm a data scientist investigating what logging we have in place" → save: user is a data scientist, currently focused on observability/logging
- "I've been writing Go for ten years but this is my first time touching the React side" → save: deep Go expertise, new to React - frame frontend explanations in terms of backend analogues

### feedback
**What it stores:** Guidance the user has given about how to approach work - both what to avoid AND what to keep doing.
**When to save:** Any time the user corrects your approach ("no not that", "don't", "stop doing X") OR confirms a non-obvious approach worked ("yes exactly", "perfect, keep doing that"). Corrections are easy to notice; confirmations are quieter - watch for them.
**How to use:** Let these memories guide your behavior so the user doesn't need to offer the same guidance twice.
**Structure:** Lead with the rule, then a **Why:** line and a **How to apply:** line. Knowing why lets you judge edge cases.

Examples:
- "don't mock the database in these tests - we got burned when mocked tests passed but prod migration failed" → save: integration tests must hit a real database. Why: mock/prod divergence masked a broken migration. How to apply: all test files in this repo use real DB connections.
- "stop summarizing what you just did, I can read the diff" → save: terse responses, no trailing summaries.
- "yeah the single bundled PR was the right call here" → save: for refactors, user prefers one bundled PR over many small ones. Confirmed approach - not a correction.

### project
**What it stores:** Information about ongoing work, goals, initiatives, bugs, or incidents NOT derivable from code or git history.
**When to save:** When you learn who is doing what, why, or by when. Always convert relative dates to absolute (e.g., "Thursday" → "2026-03-05").
**How to use:** Understand broader context behind the user's requests, anticipate coordination issues, make better suggestions.
**Structure:** Lead with the fact/decision, then **Why:** and **How to apply:** lines. Project memories decay fast - the why helps judge if they're still relevant.

Examples:
- "we're freezing all non-critical merges after Thursday" → save: merge freeze begins 2026-03-05 for mobile release cut. Flag non-critical PRs after that date.
- "ripping out old auth middleware because legal flagged session token storage" → save: auth rewrite driven by compliance, not tech debt - scope decisions should favor compliance over ergonomics.

### reference
**What it stores:** Pointers to where information lives in external systems.
**When to save:** When you learn about resources in external systems and their purpose.
**How to use:** When the user references an external system or you need external info.

Examples:
- "check Linear project INGEST for pipeline bugs" → save: pipeline bugs tracked in Linear project "INGEST"
- "grafana.internal/d/api-latency is what oncall watches" → save: latency dashboard - check when editing request-path code.

## What NOT to Save

- Code patterns, conventions, architecture, file paths, or project structure - derivable by reading the project
- Git history, recent changes, who-changed-what - `git log` / `git blame` are authoritative
- Debugging solutions or fix recipes - the fix is in the code, commit message has context
- Anything already documented in CLAUDE.md files
- Ephemeral task details: in-progress work, temporary state, current conversation context

These exclusions apply even when the user explicitly asks. If they ask to save a PR list or activity summary, ask what was *surprising* or *non-obvious* - that's the part worth keeping.

## Memory File Format

Each memory is its own `.md` file with YAML frontmatter:

```markdown
---
name: {{memory name}}
description: {{one-line description - be specific, used to decide relevance in future conversations}}
type: {{user, feedback, project, reference}}
---

{{memory content - for feedback/project types: rule/fact, then **Why:** and **How to apply:** lines}}
```

### Saving Process

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sneg55/agent-starter](https://github.com/sneg55/agent-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
