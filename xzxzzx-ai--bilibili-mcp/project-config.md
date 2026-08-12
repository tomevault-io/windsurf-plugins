---
trigger: always_on
description: This repository is `@xzxzzx/bilibili-mcp`, a TypeScript MCP server for extracting Bilibili video subtitles, metadata, and popular comments.
---

# AGENTS.md

## Project Role

This repository is `@xzxzzx/bilibili-mcp`, a TypeScript MCP server for extracting Bilibili video subtitles, metadata, and popular comments.

The expected collaboration model is:

- Codex: planning, architecture direction, task decomposition, risk review, and final verification guidance.
- Claude Code: implementation work based on the execution brief produced by Codex. The concrete model is chosen by the user or current runtime and may change over time.

Codex should usually produce a clear execution brief before implementation starts. Claude Code should execute the brief, keep changes scoped, then return the diff, logs, or summary for review.

Codex may invoke one bounded Claude Code implementation agent through the Paseo CLI after writing the execution brief. Codex owns the launch, monitoring, review, and same-scope follow-up; the user does not manually transfer prompts between tools. Do not create autonomous agent trees or broaden the approved task.

Do not hard-code a model choice in repository configuration, scripts, prompts, or handoff instructions unless the user explicitly requests that exact model configuration.

## Working Mode

Before changing code, Codex should clarify or infer:

- The update goal.
- The files and modules likely involved.
- The minimum viable change.
- The validation commands.
- The risks and rollback points.

Preferred execution brief format:

```markdown
## Update Goal
## Current Judgment
## Recommended Approach
## Things To Avoid
## Claude Code Execution Steps
## Acceptance Criteria
## Risks
```

For larger changes, split the work into small tasks that can be implemented and verified independently.

Use `docs/templates/task-ticket.md`, or a GitHub issue produced by the Matt Pocock workflow, under this standard:

- Task takes 30 minutes or less and has no public behavior change: no task ticket required; a direct Codex handoff is enough.
- Task touches multiple files, tests, security, package/release workflow, or MCP tool behavior: use a task ticket.
- Task comes from a PRD, roadmap, multi-task split, or Claude Code loop workflow: task ticket is required.

Light tickets may fill only objective, scope, files, acceptance criteria, verification, and stop/report conditions. Use the full template for broad, dependent, risky, or loop-executed work.

When a Matt GitHub issue is the task ticket, do not duplicate it as a local ticket. The Codex-to-Claude handoff must reference the issue and add any project-specific files, verification, security, rollback, and stop/report constraints that the issue does not contain.

If Codex believes Claude Code should execute a task, Codex should produce a bounded handoff, launch Claude Code through Paseo, and review the returned report and diff. Stop before launch only when the task needs a material user decision or new authority.

If implementation work raises an architectural or product decision, Claude Code should report the decision point and stop instead of guessing.

## Markdown Agent Communication

Codex and Claude Code should communicate substantial implementation work through Markdown handoffs and reports, not only through transient chat.

- Use `docs/agent-memory/agent-communication.md` as the protocol and template source.
- For substantial implementation work, Codex should create `docs/agent-memory/handoffs/YYYY-MM-DD-<topic>-codex-to-claude.md`.
- Claude Code should return a Markdown report using the template in `agent-communication.md`; if file-backed, use `docs/agent-memory/handoffs/YYYY-MM-DD-<topic>-claude-report.md`.
- Short, single-command tasks can stay in chat Markdown, but release, package, credential, MCP tool, and multi-file implementation work should use file-backed Markdown handoffs.
- Handoffs must include objective, files to inspect/edit, required capabilities, constraints, execution steps, verification commands, acceptance criteria, things not to change, and stop/report conditions.
- Reports must include files changed, commands run, command results, unresolved risks, skipped checks, and decision points.
- Reports must include a `Harness Artifacts` section covering task ticket, research note, QA checklist, codemap, harness-security, and harness-eval status.
- Do not include secrets, full Cookie values, `.env` contents, npm tokens, GitHub tokens, or private credentials in handoff/report Markdown.

### Paseo Execution

- Use the installed `paseo` CLI to run Claude Code; the user is not expected to operate Claude Code manually.
- Before each launch, read `C:\Users\ZX\.paseo\orchestration-preferences.json` and resolve the implementation provider from `providers.impl`; an explicit user choice overrides that preference. Do not hard-code a model in repository files or handoffs.
- Check Paseo availability without restarting its daemon. Never restart the daemon without explicit user approval.
- Default to one detached, bounded implementation agent in this repository. The prompt must reference the handoff file and, when applicable, the Matt GitHub Issue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XZXZZX-Ai/bilibili-mcp](https://github.com/XZXZZX-Ai/bilibili-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
