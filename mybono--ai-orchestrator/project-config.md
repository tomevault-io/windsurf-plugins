---
trigger: always_on
description: Handles small, targeted changes using the lightest available model. Intended for single-function fixes, import updates, renames, and constant additions.
---

[README](../README.md) · [Architecture](ARCHITECTURE.md) · **Agents** · [Skills & Commands](SKILLS.md) · [Cluster](CLUSTER.md)

---

# Agents

Agents are markdown files in `agents/`. Each file defines the behavior of a subagent that Claude Code can spawn. The `/implement` command orchestrates agents automatically; agents can also be invoked on demand.

## [Planner](../agents/planner.md)

Analyzes a coding task, explores the codebase, and writes `.claude/context/task_context.md`. It never writes production code.

- **Triggered by**: `/implement`, always first, before any code is written
- **Model**: Claude Sonnet (orchestrator itself — no Ollama call)
- **Ollama**: not used for planning — Claude reads files directly and writes the plan

The planner checks for `.claude/context/project_overview.md` on every run (Phase 0). If it exists, the planner runs `git status --short` and `git diff --name-only HEAD~1 HEAD` to detect stale entries — any file from `## Key Files` that appears in the output is marked [STALE] and re-read fully. Files not marked stale are trusted from the cache. If the overview does not exist, the planner does a full codebase exploration.

The output `task_context.md` contains: the task description, a step-by-step plan, exact function signatures, copy-pasted code patterns, anti-patterns to avoid, and full contents of every file that will be changed.

After writing `task_context.md`, the planner updates `.claude/context/project_overview.md` with any new architectural findings.

## [pre-reviewer]

Checks that Claude's plan complies with domain-specific standards before implementation begins. Not an architectural validator — Claude already made those decisions. A mechanical checklist against the rules loaded by triage.

- **Triggered by**: `/implement` (Step 1.5), after Claude has written the task context
- **Ollama role**: `pre-reviewer` (qwen2.5-coder:7b)

Reads `## Plan`, `## Exact Signatures`, `## Anti-patterns`, and `## Files to Change` from `task_context.md`, plus `## Domain Standards` and `## Constraints` from `triage.md`. Verifies compliance with domain rules (e.g. HEALTHCHECK syntax for docker, no hardcoded secrets for security, correct HTTP verbs for api). Writes verdict to `.claude/context/pre_review.md`. If non-compliant, Claude updates the plan before coder runs.

## [coder](../agents/coder.md)

Reads `task_context.md` and implements the changes by calling Ollama for code generation, then applies them with Edit and Write tools.

- **Triggered by**: `/implement`, after planner completes
- **Ollama role**: `coder` (model: `hf.co/bartowski/Qwen2.5-Coder-14B-Instruct-GGUF:IQ4_XS`)

The coder does not re-explore the codebase. It reads `task_context.md` and `triage.md` directly — the orchestrator passes only file paths, not content. For non-trivial generation it calls:

```bash
bash ~/.claude/call_ollama.sh --role coder --prompt-file "$TMP_PROMPT"
```

After applying changes, it runs `python3 -m py_compile` (Python) or `tsc --noEmit` (TypeScript) per changed file, and writes a structured summary to `.claude/context/coder_output.md` with sections: `## Verdict`, `## Changed Files`, `## Skipped`, `## Issues`.

## [reviewer](../agents/reviewer.md)

Reviews code diffs against project standards, runs a syntax check, and calls Ollama for logic and bug analysis. Returns `APPROVED` or `NEEDS CHANGES` with a list of issues.

- **Triggered by**: `/implement`, after coder and build check; tiered — fast then deep, per changed file, in parallel
- **Ollama role**: `reviewer` (qwen2.5-coder:7b) for deep review; `quick-coder` (qwen2.5-coder:7b) for fast review

Two-tier review per changed file:

1. **Fast review** (`quick-coder`) — syntax, style, obvious bugs. Writes `.claude/context/review_fast_<filename>.md`.
2. **Deep review** (`reviewer`) — logic, security, architecture. Runs only if fast review flagged issues or triage detected `security`/`api`/`complex` domains. Writes `.claude/context/review_deep_<filename>.md`.

Each output file has the same structure:

```markdown
## Verdict
APPROVED | NEEDS CHANGES

## Issues
- `<path>:<line>` [<Rule>] <description> — Fix: <fix>

## Notes
- <optional observations>
```

The fix loop reads only files with `Verdict: NEEDS CHANGES` and re-reviews only those files after the fix.

## [quick-coder](../agents/quick-coder.md)

Handles small, targeted changes using the lightest available model. Intended for single-function fixes, import updates, renames, and constant additions.

- **Triggered by**: user or agent request for a change under ~30 lines that does not require planning
- **Ollama role**: `quick-coder` (qwen2.5-coder:7b)

The agent follows the **Expert Committer Rules** defined in `plugins/committer/commands/commit.md`. If the task turns out to require more than one file or more than ~30 lines, quick-coder stops and recommends using `/implement` instead. Prompts must be short because the model has a 4096-token context window.

No review step follows quick-coder for trivial changes.

## [commit](../agents/commit.md)

Stages and commits all pending changes. Generates the commit message via Ollama.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mybono/ai-orchestrator](https://github.com/Mybono/ai-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
