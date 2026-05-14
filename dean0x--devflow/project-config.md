---
trigger: always_on
description: Instructions for developers and AI agents working on Devflow. For user docs, see README.md.
---

# Devflow Development Guide

Instructions for developers and AI agents working on Devflow. For user docs, see README.md.

## Purpose

Devflow enhances Claude Code with intelligent development workflows. Modifications must:
- Maintain brutal honesty in review outputs (no sycophancy)
- Preserve context across sessions
- Enhance developer empowerment without replacing judgment
- Ensure all commands are self-documenting

## Architecture Overview

Plugin marketplace with 18 plugins (9 core + 9 optional language/ecosystem), each following the Claude plugins format (`.claude-plugin/plugin.json`, `commands/`, `agents/`, `skills/`).

| Plugin | Purpose | Teams Variant |
|--------|---------|---------------|
| `devflow-implement` | Complete task implementation lifecycle | Optional |
| `devflow-plan` | Unified design planning with gap analysis | Optional |
| `devflow-code-review` | Comprehensive code review | Optional |
| `devflow-resolve` | Review issue resolution | Optional |
| `devflow-debug` | Competing hypothesis debugging | Optional |
| `devflow-explore` | Codebase exploration with KB creation | Optional |
| `devflow-self-review` | Self-review (Simplifier + Scrutinizer) | No |
| `devflow-ambient` | Ambient mode — three-tier intent classification (QUICK/GUIDED/ORCHESTRATED) | No |
| `devflow-core-skills` | Auto-activating quality enforcement | No |
| `devflow-audit-claude` | Audit CLAUDE.md files (optional) | No |
| `devflow-typescript` | TypeScript language patterns (optional) | No |
| `devflow-react` | React framework patterns (optional) | No |
| `devflow-accessibility` | Web accessibility patterns (optional) | No |
| `devflow-ui-design` | UI design patterns (optional) | No |
| `devflow-go` | Go language patterns (optional) | No |
| `devflow-python` | Python language patterns (optional) | No |
| `devflow-java` | Java language patterns (optional) | No |
| `devflow-rust` | Rust language patterns (optional) | No |

Commands with Teams Variant ship as `{name}.md` (parallel subagents) and `{name}-teams.md` (Agent Teams with debate). The installer copies the chosen variant based on `--teams`/`--no-teams` flag.

**Build-time asset distribution**: Skills and agents are stored once in `shared/skills/` and `shared/agents/`, then copied to each plugin at build time based on `plugin.json` manifests. This eliminates duplication in git.

**Working Memory**: Four shell-script hooks (`scripts/hooks/`) provide automatic session continuity. Toggleable via `devflow memory --enable/--disable/--status` or `devflow init --memory/--no-memory`. UserPromptSubmit (`prompt-capture-memory`) captures user prompt to `.memory/.pending-turns.jsonl` queue. Stop hook captures `assistant_message` (on `end_turn` only) to same queue, then spawns throttled background `claude -p --model haiku` updater (skips if triggered <2min ago; concurrent sessions serialize via mkdir-based lock). Background updater uses `mv`-based atomic handoff to process all pending turns in batch (capped at 10 most recent), with crash recovery via `.pending-turns.processing` file. Updates `.memory/WORKING-MEMORY.md` with structured sections (`## Now`, `## Progress`, `## Decisions`, `## Modified Files`, `## Context`, `## Session Log`). SessionStart hook → injects previous memory + git state as `additionalContext` on `/clear`, startup, or compact (warns if >1h stale; injects pre-compact memory snapshot when compaction happened mid-session). PreCompact hook → saves git state + WORKING-MEMORY.md snapshot + bootstraps minimal WORKING-MEMORY.md if none exists. Disabling memory removes all four hooks. Use `devflow memory --clear` to clean up pending queue files across projects. Zero-ceremony context preservation.

**Ambient Mode**: Three-layer architecture for always-on intent classification. SessionStart hook (`session-start-classification`) reads lean classification rules (`~/.claude/skills/devflow:router/classification-rules.md`, ~30 lines) and injects as `additionalContext` — once per session, deterministic, zero model overhead. UserPromptSubmit hook (`preamble`) injects a one-sentence prompt per message triggering classification + conditional router loading via Skill tool. Router SKILL.md is a pure skill lookup table (~50 lines) loaded on-demand only for GUIDED/ORCHESTRATED depth — maps intent×depth to domain and orchestration skills. Toggleable via `devflow ambient --enable/--disable/--status` or `devflow init`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dean0x/devflow](https://github.com/dean0x/devflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
