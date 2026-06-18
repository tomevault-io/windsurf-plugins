---
trigger: always_on
description: Bootstrap or audit a serious agent-led engineering workflow in any repository. Use when Codex must establish or update project canon, PRD-first execution, GitHub Epic/Task/Bug decomposition, PR verification discipline, multi-agent release coordination, reusable bootstrap files, or a model-agnostic workflow that future GPT/Codex and Claude-style agents can reuse without re-explaining the process in chat.
---


# Engineering Kernel

Use this skill when the task is to establish or upgrade a repository's engineering operating system, not when the task is simply to ship a normal feature.

Read [ENGINEERING_KERNEL.yaml](ENGINEERING_KERNEL.yaml) for the machine-readable core.
Read [references/BOOTSTRAP.md](references/BOOTSTRAP.md) when you need to materialize the kernel into a project.
Read [references/MODEL_ADAPTERS.md](references/MODEL_ADAPTERS.md) when the user asks how the kernel should map across GPT/Codex and Claude-style agents.
Read [references/BEHAVIORAL_OVERLAY.md](references/BEHAVIORAL_OVERLAY.md) when the user asks whether a thin behavior-only guideline layer should exist across `CLAUDE.md`, Cursor rules, or skill/plugin surfaces.
Read [references/SUPERPOWERS_SKILL_ORCHESTRATION.md](references/SUPERPOWERS_SKILL_ORCHESTRATION.md) when the user asks how Superpowers or another process-skill pack should be used with the kernel.
Read [references/AGENTIC_CODING_ORCHESTRATION.md](references/AGENTIC_CODING_ORCHESTRATION.md) when the user asks how orchestrator/worker/reviewer agentic coding should be structured, limited, reviewed, and recovered after resource failures.
Read [references/MULTI_AGENT_RELEASE_COORDINATION.md](references/MULTI_AGENT_RELEASE_COORDINATION.md) when the user asks how multiple agents, branches, worktrees, integration PRs, staging/prod deploys, runtime recovery, or handoffs should be coordinated in one production-bound project.
Read [references/PROJECT_LOCAL_WORKERS.md](references/PROJECT_LOCAL_WORKERS.md) when the user asks how to create global fallback workers, project-local workers, no-MCP coding/review/docs workers, rich-MCP orchestrators, or worker selection rules.
Read [references/MCP_TOOLING.md](references/MCP_TOOLING.md) when the user asks how MCP servers, App connector tooling, GitHub App permissions, or local `gh` fallback should be used.
Read [references/RESEARCH_POLICY.md](references/RESEARCH_POLICY.md) when the user asks how research and evidence collection should work.
Read [references/EXECUTION_SURFACES.md](references/EXECUTION_SURFACES.md) when the user asks where work should run locally versus in GitHub Actions or CI.
Read [references/ENVIRONMENT_PROMOTION.md](references/ENVIRONMENT_PROMOTION.md) when the user asks how local, verify, staging, and production environments should be separated and promoted safely.
Read [references/KERNEL_SYNC_POLICY.md](references/KERNEL_SYNC_POLICY.md) when the user asks how live project learnings should be reviewed and promoted back into the universal kernel.
Read [references/SESSION_ISSUE_SYNC.md](references/SESSION_ISSUE_SYNC.md) when the user asks how serious slices should update or explicitly skip GitHub issue state at closeout.
Read [references/WORK_ITEM_ROUTING.md](references/WORK_ITEM_ROUTING.md) when the user asks where work items, issues, backlog tasks, or cross-repo follow-ups should live.
Read [references/PROJECT_HEALTH_AUDIT.md](references/PROJECT_HEALTH_AUDIT.md) when the user asks how to audit a project for missing operating artifacts such as SSOT, decision log, escalation rules, incident log, or eval cases.
Read [references/OPTIONAL_WEEKLY_OPERATING_LOOP.md](references/OPTIONAL_WEEKLY_OPERATING_LOOP.md) when the user asks how optional weekly planning, outcomes, retro scorecards, or carryover decisions should work.
Read [references/KERNEL_UPSTREAM_AWARENESS.md](references/KERNEL_UPSTREAM_AWARENESS.md) when the user asks how consumer projects should notice upstream kernel changes and decide whether to adopt them.
Read [references/KERNEL_ADOPTION_TASK.md](references/KERNEL_ADOPTION_TASK.md) when the user asks what exact downstream `Task` should be opened or updated after `kernel_upstream_check` reports drift.
Read [references/KERNEL_FLEET_SWEEP.md](references/KERNEL_FLEET_SWEEP.md) when the user asks how one operator machine should check kernel drift across many consumer repositories at once.
Read [references/GITHUB_DELIVERY.md](references/GITHUB_DELIVERY.md) when the user asks how branch/PR/merge flow should work end-to-end.
Read [references/BUG_INTAKE.md](references/BUG_INTAKE.md) when the user asks how runtime failures should become GitHub `Bug` issues without noisy duplication.

The bug-intake rule is explicit:

- do not create bug issues from raw logs or chat alerts
- use one stable fingerprint per bug class
- update the existing open bug issue when the fingerprint matches

The session issue sync rule is explicit:

- serious closeouts record `Issue Sync: updated | skipped | not_applicable`
- durable status, next steps, verification, and links belong in the issue body
- comments are for short chronological notes, explicit user requests, or external blockers

The operational canon rules are explicit:

- `Work Item Routing` decides the target repo or project-local surface before issue creation; never use the current checkout as the implicit target

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexxety/agent-engineering-kernel](https://github.com/alexxety/agent-engineering-kernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
