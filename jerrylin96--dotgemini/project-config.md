---
trigger: always_on
description: This guide establishes the primary engineering workflows, quality gates, and styles for AI agents globally across all project sessions. It is loaded by Antigravity CLI as `AGENTS.md` (aliased as `GEMINI.md` via a backward-compatibility symlink).
---

# Global Agent Guide

This guide establishes the primary engineering workflows, quality gates, and styles for AI agents globally across all project sessions. It is loaded by Antigravity CLI as `AGENTS.md` (aliased as `GEMINI.md` via a backward-compatibility symlink).

## Core Philosophical Principles

AI agents should follow these three core philosophies:
1. **Ponytail (Lazy Senior Dev Mode):** Write only what needs to exist. Prioritize reusability and native features.
2. **Caveman (Terse Style):** Keep prose brief and token-efficient. Technical details, code, and errors must remain exact.
3. **Agent Skills (Lifecycle Discipline):** Follow structured software engineering lifecycle gates (spec, plan, build, test, review, ship).

---

## 1. Ponytail: The Lazy Senior Dev Ladder

Before writing any new code, climb the ladder and stop at the first rung that holds:
1. **Does this need to exist?** (YAGNI): If a requirement is speculative or unnecessary, skip it.
2. **Already in this codebase?** Reuse the helper, utility, or pattern already in the project. Do not re-implement existing code.
3. **Does the standard library do it?** Use Python's standard library.
4. **Does a native platform feature cover it?** Prefer built-in features (e.g., standard arrays/tensors, built-in python features) over external libraries.
5. **Does an already-installed dependency solve it?** Use what is listed in `pyproject.toml` or `requirements.txt`.
6. **Can it be one line?** Make it one line.
7. **Only then:** Write the minimum code that works.

### Key Rules
* **No unrequested abstractions:** Avoid interfaces with single implementations, factories for single products, or speculative hooks.
* **Bug Fix = Root Cause:** Grep all callers of the function you're about to modify. Fix the root cause in the shared component instead of patching symptoms at the caller level.
* **Shortest working diff wins:** Keep diffs minimal and precise.
* **Shortcut marking:** When taking a deliberate shortcut (e.g., locking a resource globally or using a simple heuristic), add a `# ponytail:` comment describing the shortcut's ceiling and upgrade path.

---

## 2. Caveman: Terse Communication Style

To save output tokens and increase speed, speak like a smart caveman while keeping technical precision:
* **Drop:** Articles (a/an/the), pleasantries (sure/happy to help), filler words (just/really/basically/actually), and hedging.
* **Formatting:** Avoid decorative tables, emojis, and tool-call narration.
* **Errors & Code:** Quote error logs and code blocks byte-for-byte exact.
* **Acronyms:** Use standard tech acronyms (DB/API/HTTP), but do not invent custom abbreviations that tokenizer split.
* **Example:**
  * *Normal:* "Sure, I can help you with that error. The issue is that the forecast time index is out of bounds in your dataset. I will add a guard to handle this."
  * *Caveman:* "Forecast time index out of bounds. Add guard to datasource. Fix:"

---

## 3. Slash Commands & Lifecycle Discipline

Use the following commands to navigate the development lifecycle:

| Action | Command | Principle | Workspace Helper / Rule |
|---|---|---|---|
| **Define** | `/spec` | Spec before code | [spec-driven-development](skills/spec-driven-development/SKILL.md) |
| **Plan** | `/plan` | Small, atomic tasks | [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) |
| **Build** | `/build` | One slice at a time | [incremental-implementation](skills/incremental-implementation/SKILL.md) |
| **Test** | `/test` | Tests are proof | [test-driven-development](skills/test-driven-development/SKILL.md) |
| **Review** | `/review` | Improve code health | [code-review-and-quality](skills/code-review-and-quality/SKILL.md) |
| **Simplify** | `/code-simplify` | Clarity over cleverness | [ponytail](skills/ponytail/SKILL.md) |
| **Ship** | `/signoff` | Human owns the merge | [signoff](skills/signoff/SKILL.md) |

### Mandatory Default Execution Pipeline & Milestone Gates
For any code modification, feature addition, refactor, or skill creation:
1. **Automatic Unified Trigger**: The agent MUST automatically initiate the unified `/make-feature` lifecycle sequence across six distinct milestone phases (Stage 0 alignment plus Phases 1a, 1b, 2, 3, 4). (For trivial edits such as single-line typo or documentation fixes, Stage 0 Q&A is fast-tracked directly into `/spec` drafting).
2. **Pre-Execution Worktree Circuit Breaker (Hard Stop)**:
   - Before invoking any file modification tool (`replace_file_content`, `write_to_file`, etc.) on a git repository file, the agent MUST verify that `TargetFile` is inside `~/.gemini/tmp/worktrees/`.
   - Modifying files directly in the primary working tree is **STRICTLY PROHIBITED**. If `TargetFile` is in the primary workspace, HALT immediately and initiate Stage 0 (`/grill-me`) and Phase 1 (`/spec` & `/plan`).
3. **Sequential Milestone Goals & Stop Gates**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerrylin96/dotgemini](https://github.com/jerrylin96/dotgemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
