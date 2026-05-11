---
trigger: always_on
description: [6~## AGENTS.md — LLMC Agent Charter
---

[6~## AGENTS.md — LLMC Agent Charter

The user is **Dave**.

- **Repo root:** `~/src/llmc` (aka `/path/to/your/llmc/repo`)
- **Rule:** NO RANDOM CRAP IN THE REPO ROOT.  
  - If you need a scratch script, prefer `./.trash/`.
  - If it belongs in repo root by common best practice (README, pyproject, etc.), that’s fine.

---

## 1. Purpose

This file is the primary **behavioral** contract for all agents working in this repo.

- **AGENTS.md** → how to behave and work.
- **CONTRACTS.md** → environment, tooling, and policy.

If you only read one doc before acting, read **this one**, then skim **CONTRACTS.md**.

---

## 2. Agent Profile (ALL AGENTS)

### Rules of Thumb

- After changing code, run a **smoke test** before responding that the ask is successful.
- When Dave says “run tests” / “execute tests”, run the tests immediately (≤30 seconds of prep).
- Follow **GitHub best practices**:
  - Create a feature branch before non-trivial work.
  - Keep commits small and focused; prefer PR-ready patch sets.
- Before performing a rollback, enumerate every file that will change and obtain explicit approval.
- Prefer **patch-style changes** (diffs) over rewriting whole files.

### Git Safety Rules (CRITICAL)
This is a multi user multi agent repo.  If you are told to commit and there
are untracked files (and there almost always will be), stop and ask what to do. 
Generally it's fine to just commit all untracked files, it's never ok to
revert or do anything that will destroy untracked files without the word
ENGAGE from the user.

- **NEVER** run `git reset HEAD~` or `git revert HEAD` without explicit approval.
- **NEVER** delete files (via `rm`, `git rm`, or any other method) without explicit approval.
- **NEVER** assume a file is "safe to delete" - always ask first.
- If you need to undo a commit, **ask Dave** and enumerate exactly what will change.
- If you see untracked files you didn't create, **ask Dave** before touching them.

---

## 3. Engineering Workflow — “The Dave Protocol”

For any task that is **Significant**  
(>1 file, non-trivial refactor, core pipeline, or anything Dave labels “Important”):

1. **Logic Gate**
   - Decide: **Small** (just do it) vs **Significant** (follow this loop).
   - If unsure, treat it as **Significant**.

2. **Overview**
   - Briefly restate the goal in your own words to confirm alignment.

3. **Imaginative / Research Phase**
   - Explore approaches, read code, RAG, docs.
   - **Do not** write implementation code yet.
   - Call out key risks / unknowns.

4. **HLD – High Level Design**
   - Describe architecture, data flow, and **test strategy**.
   - Identify which modules / services will change.
   - Get explicit approval before proceeding.

5. **SDD – Software Design Document**
   - Specify function signatures, data contracts, and concrete **test cases**.
   - Note any migrations, config changes, or CLI impacts.
   - Get approval before coding.

6. **Implementation – TDD where practical**
   - Write failing tests from the SDD (when reasonable).
   - Implement code to make tests pass.
   - Keep changeset focused and well-diffed.

7. **Verification**
   - Run targeted tests (unit / integration / CLI) for affected areas.
   - Summarize what you ran and the results.

8. **Documentation**
   - Update or create docs as needed:
     - Roadmap entries
     - Architecture docs
     - CLI usage / examples

This loop exists to prevent “cowboy coding” and keep LLMC maintainable.

---

## 4. Context Retrieval Protocol (RAG / MCP)

The repo includes a RAG system with CLI entrypoints. Use it, but don’t worship it.

### 4.1 RAG-First Contract

- **Default:** For repo/code questions, try **RAG tools first**.
- If RAG fails (no results, tool errors, or obviously weird hits), **silently fall back** to:
  - `rg` / `grep`
  - AST / structural search
  - Direct file reads
- Don’t give up after a single RAG miss:
  - Try one improved query, then fall back.
  - Never loop endlessly tweaking thresholds.

### 4.2 What RAG Scores Mean (and Don’t)

- Similarity scores from RAG are **for ranking only**.
- They are **not calibrated confidence** and are **not percentages**.
- **Never** say “this is 80% relevant” based on a raw score.
- Treat the **ordering** of results as useful; treat the **absolute number** as noisy.

### 4.3 Dependency Analysis Protocol

When you need to understand **file dependencies** (parents/children) and RAG is insufficient:

1.  **Parent Relationships (Who imports X?):**
    - Use `search_file_content` to find imports of the target module.
    - **Pattern:** `import <module>` or `from <module> import`
    - **Scope:** Search relevant directories (e.g., `src/`, `scripts/`, `tests/`) or the whole repo if unsure.
    - **Example:** `search_file_content --pattern "from scripts.router import" --include "*.py"`

2.  **Child Relationships (Who does X import?):**
    - Use `read_file` on the target file.
    - Analyze the `import` and `from ... import` statements at the top of the file.

**Do not** rely on external enrichment tools for this unless explicitly instructed.

---

## 5. Startup Context (FIRST THING TO RUN)

**Before diving into any task**, run `mcschema` to get instant codebase orientation:

```bash
llmc-cli run mcschema schema
```

This gives you (~600 tokens):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vmlinuzx/llmc](https://github.com/vmlinuzx/llmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
