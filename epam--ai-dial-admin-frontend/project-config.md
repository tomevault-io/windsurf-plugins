---
trigger: always_on
description: Interactively create GitHub issues (Bug, Feature, Task) for the current repository. Infrastructure changes are Tasks auto-labeled `infra-task`. Parses optional args for type and description, asks targeted questions, assigns labels, and creates the issue via gh CLI.
---


Create a GitHub issue in this repository. Guides the user through an interactive flow to gather all required information, then creates the issue via `gh issue create`.

## Prerequisites

Before starting the flow, verify `gh` CLI is available and authenticated:

```bash
gh auth status
```

- If `gh` is not installed → stop and tell the user: "GitHub CLI (`gh`) is required. Install it: https://cli.github.com/"
- If not authenticated → stop and tell the user: "Run `gh auth login` to authenticate first."
- If authenticated → proceed to Step 0

**Usage**: `/create-ticket [type: description]`

Examples:
- `/create-ticket` — fully interactive
- `/create-ticket bug: login page crashes on Safari`
- `/create-ticket feature: add bulk delete for models`
- `/create-ticket task: refactor Sidebar for reuse in entity lists`
- `/create-ticket infra: add LOG_LEVEL to prod`

---

## Step 0 — Parse Args

If the user provided arguments after `/create-ticket`, parse them:

- Look for a type prefix: `bug:`, `feature:`, `task:`, `infra:` (case-insensitive)
  - `bug:` → Bug
  - `feature:` → Feature
  - `task:` → Task (general engineering)
  - `infra:` → Task + force `infra-task` label (shortcut for infra change)
  - If found → set issue type, use the rest as a seed for the title/description
  - If not found → treat the entire arg string as a description seed and ask for type
- If no args at all → start fully interactive from Step 1

When args provide a description seed, use it to:
- **Generate a proposed title** — concise, under 80 characters
- **Generate a proposed summary** — a short paragraph followed by key points as a bullet list. For bugs, also draft the actual/expected result from context.
- Pre-fill relevant body fields where obvious
- Auto-assign labels ONLY when the description makes them 100% clear
- For ANY label where assignment is not certain from the description, ask the user directly

---

## Step 1 — Issue Type

Skip if parsed from args.

Use **AskUserQuestion** to ask:
> "What type of issue do you want to create?"

Options (these match the repo's GitHub issue types):
- **Bug** — Report a problem or defect
- **Feature** — Request a new feature or enhancement
- **Task** — Engineering work that is not a user-facing feature (refactor, reuse, tech debt, cleanup, test/build improvements, AND infrastructure changes — env var, secret, config, deployment setting)

Infrastructure changes are NOT a separate top-level type. They're a Task that gets auto-labeled `infra-task`. Detection happens in Step 3 from context keywords (env var, secret, prod/uat, deployment, config, LOG_LEVEL, etc.).

---

## Step 2 — Title & Summary Proposal

### 2a. Optional Code Research

Before drafting the proposal, check whether the user's input references code concepts that would benefit from codebase investigation.

**Trigger signals** (offer the question when ANY are present):
- Verbs: refactor, reuse, extract, consolidate, migrate, rename, split, merge, deduplicate, unify, abstract, move, replace
- Implementation nouns paired with an action: component, hook, utility, service, module, layout, route, page, store, context, provider
- Explicit file paths, component names (e.g., `Sidebar`), or function/hook names

If triggered, use **AskUserQuestion**:
> "Your request touches code (e.g., refactoring/reuse). Do you want me to dive into the codebase and include relevant details in the ticket?"

Options:
- **Yes, research** — Explore the codebase and add findings to the description
- **No, skip** — Proceed without code research

If **Yes**, spawn an **Explore** agent (via the Agent tool with `subagent_type: Explore`) to find:
- Current location(s) of the subject code (file paths with line references)
- Similar or duplicated patterns elsewhere in the codebase
- Files/components/consumers that would be affected by the change
- Existing conventions or abstractions to align with

Keep the summary concise — bullet points with `path:line` references, not prose walls. These findings feed into **2b** (title sharpness, description clarity, and a new **Details** section in the body).

### 2b. Draft the Proposal

Based on the user's input, answers so far, and any code research findings:

1. **Propose a title** — concise, under 80 characters. If research was done, make it specific (e.g., "Refactor `Sidebar` for reuse in `ModelList`, `AdapterList`, `UserList`").
2. **Propose a summary** — structured as:
   - A short description paragraph (1-3 sentences explaining *what* and *why*)
   - **Key points** as a bullet list extracting the important details
   - For **bugs**: also include drafted "Actual result" and "Expected result" if enough context exists
   - If code research was done: a **Details** section listing current location, affected files, duplicated patterns, and recommended scope (with `path:line` references)

3. **List all labels** that will be auto-assigned based on the issue type and any labels already determined from context (e.g., `bug`, `Design Required` if obvious from description). Mark labels that will be asked about later as "TBD".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epam/ai-dial-admin-frontend](https://github.com/epam/ai-dial-admin-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
