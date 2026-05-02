---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Code skill plugin** (`long-task-agent`) enabling multi-session execution of complex software projects. Implements: Requirements → UCD → Design → ATS → Init → Worker → ST → Finalize, with Hotfix and Increment re-entry points. State bridges via on-disk artifacts. 14 skills loaded on-demand via the `Skill` tool; bootstrap router (`using-long-task`) routes to the correct phase based on project state. Standalone `/deep-explore` skill for on-demand codebase exploration.

## Key Commands

> **Path note**: paths below are consumer-facing (`long-task-agent/...`). In this repo, replace `long-task-agent/` with `./` or omit.

| Purpose | Command |
|---------|---------|
| Init project | `python scripts/init_project.py <name> --path <dir> [--lang python\|java\|typescript] [--line-cov N] [--branch-cov N] [--mutation-score N]` |
| Validate feature-list | `python scripts/validate_features.py feature-list.json` |
| Validate guide | `python scripts/validate_guide.py long-task-guide.md [--feature-list feature-list.json]` |
| Check configs | `python scripts/check_configs.py feature-list.json [--feature N]` |
| Check DevTools MCP | `python scripts/check_devtools.py feature-list.json [--feature N]` |
| Check Jinja2 | `python scripts/check_jinja2.py [--quiet]` |
| Check MCP providers | `python scripts/check_mcp_providers.py tool-bindings.json [--feature N]` |
| Apply tool bindings | `python scripts/apply_tool_bindings.py tool-bindings.json [--defaults\|--regenerate-defaults\|--dry-run]` |
| Validate ATS | `python scripts/validate_ats.py docs/plans/ats.md [--srs docs/plans/srs.md]` |
| Check ATS coverage | `python scripts/check_ats_coverage.py docs/plans/ats.md --feature-list feature-list.json [--feature N] [--strict]` |
| Check ST readiness | `python scripts/check_st_readiness.py feature-list.json` |
| Validate ST cases | `python scripts/validate_st_cases.py docs/test-cases/feature-N.md [--feature-list feature-list.json --feature N]` |
| Validate increment | `python scripts/validate_increment_request.py increment-request.json` |
| Validate bugfix | `python scripts/validate_bugfix_request.py bugfix-request.json` |
| Get tool commands | `python scripts/get_tool_commands.py feature-list.json [--json] [--bindings tool-bindings.json]` |
| Check real tests | `python scripts/check_real_tests.py feature-list.json [--feature N] [--require-for-deps] [--json]` |
| Check retro auth | `python scripts/check_retro_auth.py feature-list.json` |
| Validate retro record | `python scripts/validate_retrospective_record.py docs/retrospectives/record.md` |
| Check retro readiness | `python scripts/check_retrospective_readiness.py` |
| Post retro report | `python scripts/post_retrospective_report.py --feature-list feature-list.json` |
| Run all tests | `python -m pytest tests/` |
| Run single test | `python -m pytest tests/test_<script_name>.py` |
| Auto-loop (Claude Code) | `python scripts/auto_loop.py feature-list.json [--max-iterations 30] [--log-dir logs] [--cooldown 10]` |
| Auto-loop (OpenCode) | `python scripts/auto_loop_opencode.py feature-list.json [--model anthropic/claude-sonnet-4-6]` |
| Deep-explore codebase | Invoke `long-task:long-task-explore` or `/deep-explore [quick\|standard\|deep] [--focus area] [--path dir]` |

## Architecture

### 14-Skill System

#### Phase Skills

| Skill | Phase | Trigger |
|-------|-------|---------|
| `using-long-task` | Bootstrap | Routes to correct phase; invoked by LLM at session start based on skill description |
| `long-task-hotfix` | Hotfix | `bugfix-request.json` exists (HIGHEST priority) |
| `long-task-increment` | Phase 1.5 | `increment-request.json` exists |
| `codebase-scanner` (SubAgent) | Phase 0-pre | No SRS/rules docs, >3 source files — brownfield scan |
| `long-task-requirements` | Phase 0a | No SRS, no design doc, no feature-list.json |
| `long-task-ucd` | Phase 0b | SRS exists, no UCD/design doc, no feature-list.json |
| `long-task-design` | Phase 0c | SRS + UCD exist (or no UI), no design doc, no feature-list.json |
| `long-task-ats` | Phase 0d | Design doc exists, no ATS doc, no feature-list.json |
| `long-task-init` | Phase 1 | ATS doc exists (or auto-skipped), no feature-list.json |
| `long-task-work` | Phase 2 | feature-list.json exists, some active features failing |
| `long-task-st` | Phase 3 | feature-list.json exists, ALL active features passing |

#### Standalone Skills (no pipeline dependency)

| Skill | Purpose | Trigger |
|-------|---------|---------|
| `long-task-explore` | Deep codebase exploration — architecture, data flow, domain model, API surface, dependencies, code health | On-demand via `/deep-explore` |

#### Discipline Skills (sub-skills of long-task-work)

| Skill | Purpose |
|-------|---------|
| `long-task-feature-design` | Feature Detailed Design — interface contracts, pseudocode, diagrams, test inventory |
| `long-task-tdd` | TDD Red-Green-Refactor |
| `long-task-quality` | Coverage Gate + Feature-Scoped Mutation Gate |
| `long-task-feature-st` | Black-Box Feature Acceptance Testing (self-managed lifecycle, Chrome DevTools MCP + ISO/IEC/IEEE 29119) |

#### Meta Skills


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suriyel/longtaskforagent](https://github.com/suriyel/longtaskforagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
