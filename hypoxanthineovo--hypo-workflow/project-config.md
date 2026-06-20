---
trigger: always_on
description: Run a serialized prompt execution pipeline from a local `.pipeline/` workspace. Use this skill whenever the user says "开始执行", "继续 pipeline", "执行下一步", "pipeline status", "跳过当前步骤", "skip step", "中止", "abort", or invokes `/hw:start`, `/hw:resume`, `/hw:status`, `/hw:skip`, `/hw:stop`, `/hw:report`, `/hw:chat`, `/hw:analysis`, `/hw:plan`, `/hw:plan:deep`, `/hw:plan:extend`, `/hw:plan:review`, `/hw:cycle`, `/hw:accept`, `/hw:reject`, `/hw:explore`, `/hw:sync`, `/hw:maintain`, `/hw:docs`, `/hw:patch`,
---


# Hypo-Workflow v13.1.0-beta.1

> **Claude Code 用户**：安装/启用 `hw` plugin 后使用 `/hw:*` 指令；Claude 原生 `/resume` 不属于 Hypo-Workflow，Hypo 恢复命令始终是 `/hw:resume`。
>
> **Codex 用户**：本文件是完整的 Skill 入口，继续使用 `/hw:*` 指令。

## Role of this file

This SKILL.md serves as the **index and routing summary** for Hypo-Workflow. For the complete execution specification of each command, refer to its dedicated skill file under `skills/*/SKILL.md`. The sections below provide:
- A command routing table
- Global rules (output language, plan discipline, analysis boundary)
- Cross-cutting conventions

**Do not edit behavior here that exists in the dedicated skill files — keep them in sync, or prefer the dedicated file.**

## 指令列表

| Command | Description |
|---------|-------------|
| `/hw:start` | Initialize and start the pipeline from the first prompt |
| `/hw:resume` | Resume from the last interrupted state |
| `/hw:status` | Show current pipeline progress; use `--full` to bypass compact context |
| `/hw:skip` | Skip the current prompt and advance |
| `/hw:stop` | Gracefully stop and save state |
| `/hw:report` | Show compact report summaries, latest scores, or `--view <M>` full report |
| `/hw:chat` | Enter lightweight append conversation mode |
| `/hw:analysis` | Enter, continue, end, or report an interactive Analysis investigation lane |
| `/hw-analysis` | OpenCode native alias for `/hw:analysis` |
| `/hw:plan` | Enter Plan Mode through `plan/PLAN-SKILL.md` |
| `/hw:plan:deep` | Maintain Deep Plan discussion packages before ordinary Plan conversion |
| `/hw:plan:discover` | Run the Discover phase of Plan Mode |
| `/hw:plan:decompose` | Run the Decompose phase of Plan Mode |
| `/hw:plan:generate` | Run the Generate phase of Plan Mode |
| `/hw:plan:confirm` | Run the Confirm phase of Plan Mode |
| `/hw:plan:extend` | Append milestones to an active Cycle |
| `/hw:plan:review` | Run Plan Review for the current or all milestones |
| `/hw:cycle` | Create, list, view, close, and archive delivery Cycles |
| `/hw:accept` | Accept pending Cycle work and complete the manual acceptance gate |
| `/hw:reject` | Reject pending Cycle work with structured feedback and reopen the Cycle |
| `/hw:explore` | Start an isolated exploration worktree and record exploration metadata |
| `/hw:sync` | Synchronize project adapters and lightweight derived context without executing pipeline milestones |
| `/hw:maintain` | Manage maintenance queue, ledger, evidence, backups, and gated maintenance operations |
| `/hw:maintain status` | Show maintenance queue and ledger status |
| `/hw:maintain scan` | Create scan evidence for maintenance candidates |
| `/hw:maintain plan` | Plan maintenance operations as queue items |
| `/hw:maintain queue` | Inspect or update the maintenance queue |
| `/hw:maintain run` | Run approved maintenance operations |
| `/hw:maintain apply` | Apply confirmed maintenance writes |
| `/hw:maintain verify` | Verify maintenance apply results |
| `/hw:maintain log` | Read the sanitized maintenance ledger |
| `/hw:docs` | Generate, check, repair, and sync documentation |
| `/hw:patch` | Create, list, close, and `fix` persistent lightweight Patches |
| `/hw:patch fix` | Execute the lightweight six-step Patch repair lane |
| `/hw:pr` | Inspect, review, fix, merge, or close existing GitHub PRs / GitLab MRs through local archives |
| `/hw:pr create` | Guide GitHub PR / GitLab MR creation from existing worktree changes or a plan-first PR-sized task |
| `/hw:explain` | Answer code, config, command, or recent-change questions with cited local evidence |
| `/hw:compact` | Generate `.compact` context views for large runtime files |
| `/hw:knowledge` | Inspect Knowledge Ledger records, indexes, compact summaries, and secret references |
| `/hw:guide` | Start an interactive guide that recommends the next command path |
| `/hw:showcase` | Generate project intro docs, technical docs, slides, and an optional poster |
| `/hw:rules` | Manage rule severities, custom natural-language rules, lifecycle hooks, and rule packs |
| `/hw:init` | Initialize or rescan `.pipeline/` with architecture-aware project discovery |
| `/hw:check` | Run pipeline health checks for config, state, prompts, Notion, and architecture |
| `/hw:audit` | Run Intake-first preventive engineering audits and emit graded findings with report output |
| `/hw:quality` | Generate evidence-backed quality scorecards, baselines, comparisons, reviews, and action queues |
| `/hw:optimize` | Run an Audit+Quality guided optimization loop with backup, correctness, budget, and validation gates |
| `/hw:release` | Run the automated release flow with regression, versioning, changelog, and git steps |
| `/hw:debug` | Run symptom-driven debugging with hypotheses, validation, and optional auto-fix |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HypoxanthineOvO/Hypo-Workflow](https://github.com/HypoxanthineOvO/Hypo-Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
