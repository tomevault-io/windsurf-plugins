---
trigger: always_on
description: Code learning and audit framework. Analyze any codebase — new, legacy, or AI-generated — and produce educational explanations or architectural audits. Use when the user wants to understand WHAT and WHY behind any code, not just accept it.
---


# AntiVibe - Code Learning & Audit Framework

## Purpose

AntiVibe generates **learning-focused explanations or architectural audits** of any code — AI-generated, legacy, or otherwise. It helps developers understand:
- **What** the code does (functionality)
- **Why** it was written this way (design decisions)
- **When** to use these patterns (context)
- **What alternatives** exist (broader knowledge)

Works on any codebase — you don't need recent git history or AI-authored files.

## When to Use

Use AntiVibe when:
1. **Manual invocation**: User types `/antivibe` or "deep dive"
2. **Post-task learning**: After a feature/phase completes, user wants to learn from it
3. **Legacy codebases**: User wants to understand existing code they didn't write
4. **Proactive**: User says "explain what AI wrote", "walk me through", "audit this", or points at a file/directory

## What AntiVibe Produces

Output saved to `deep-dive/` folder as markdown:

```
deep-dive/
├── auth-system-2026-01-15.md
├── api-layer-2026-01-15.md
└── database-models-2026-01-15.md
```

The exact sections depend on the output mode (see [Output Mode](#output-mode)):

| Section | `compact` (default) | `full` |
|---------|:---:|:---:|
| **Overview** — what the code does and why it exists | ✅ | ✅ |
| **Key Components / Concepts** — design patterns, algorithms, CS concepts used | ✅ | ✅ |
| **Code Walkthrough** — file-by-file, line-by-line notes | — | ✅ |
| **Learning Resources** — curated docs, tutorials, videos | — | ✅ |
| **Related Code** — links to other files in the codebase | — | ✅ |

## Configuration

### Known Concepts (Skip List)

Concepts listed here will not be explained in full — the explainer will only note that they were used and in what context. Edit this list to match your current knowledge.

```yaml
known_concepts:
  - async/await
  - React hooks
  - REST APIs
```

### Output Mode

Controls how much detail is generated per run. Default is `compact` to keep token costs low.

```yaml
output_mode: compact
```

| Mode | What's included |
|------|----------------|
| `compact` (default) | Overview, key components (function-level, one line each), concepts (what + why only). No resources. No line-by-line. Max 5 files. |
| `full` | Everything in compact, plus: line-by-line walkthrough, prerequisites, curated resources, Next Steps. |

Override inline in your request:
- `"/antivibe full"`, `"full deep dive"`, `"include resources"` → `full` mode
- Default: `compact`

### Default Skill Level

Sets the explanation depth when no level is specified in the request. Options: `junior`, `mid`, `senior`. Default: `mid`.

```yaml
default_level: mid
```

| Level | Behavior |
|-------|----------|
| `junior` | Define all terms. Use analogies. Explain language features. Show full code snippets with inline comments. |
| `mid` | Skip basics. Focus on design decisions and trade-offs. Brief code references only. |
| `senior` | Skip obvious patterns. Focus only on non-obvious choices, edge cases, and architectural trade-offs. |

Level can also be specified inline in the request:
- `"explain for a junior"`, `"I'm new to this"` → `junior`
- `"I know the basics"`, `"mid level"` → `mid`
- `"senior mode"`, `"skip the basics"`, `"just the trade-offs"` → `senior`

---

## Workflow

### Step 0: Apply User Configuration
Before analyzing, read the configuration above:
- Load the `known_concepts` skip list. Any concept in this list will be acknowledged in one sentence instead of fully explained.
- Detect the skill level: check the user's request first (inline phrases take priority), then fall back to `default_level`. Apply this level consistently throughout the entire output.
- If level = `senior`, route to `agents/auditor.md` instead of continuing this workflow.

### Step 1: Identify Code to Analyze

Use the first applicable mode:

1. **Explicit** — User named specific files, a directory, or a module in their request → use those directly. No git needed. Example: "explain `src/auth/`" or "walk me through `api/routes.py`".

2. **Recent** — No explicit target given, project is a git repo, and `git diff HEAD` has output → use those changed files (current behavior for post-AI-task learning).

3. **Scan** — No explicit target, no usable git diff (legacy project, no recent changes, or not a git repo) → ask the user: "Which file, directory, or module would you like to analyze?" Do not attempt to guess.

> The code does not need to be AI-generated. AntiVibe analyzes any code.

### Step 2: Analyze Code Structure
For each file:
- Identify main purpose and responsibilities
- Note key functions, classes, modules
- Identify design patterns used (factory, singleton, observer, etc.)
- Find any complex logic or algorithms

### Step 3: Explain Concepts
For each concept/pattern found:
- **What**: Plain-language explanation
- **Why**: Why this approach was chosen over alternatives
- **When**: When to use this pattern (with context)
- **Alternatives**: Other approaches and trade-offs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohi-devhub/antivibe](https://github.com/mohi-devhub/antivibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
