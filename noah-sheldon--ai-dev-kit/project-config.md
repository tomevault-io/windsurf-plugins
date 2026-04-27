---
trigger: always_on
description: This is a production-ready AI dev kit for Python, TypeScript, web, AI/ML, and
---

# AI Dev Kit — Agent Instructions

This is a production-ready AI dev kit for Python, TypeScript, web, AI/ML, and
infrastructure workflows.

**Version:** 1.1.0

---

## Core Principles

1. **Agent-first** — delegate domain work to the right specialist. Don't generalize when a specialist exists.
2. **Test-driven** — write tests before implementation when behavior changes. RED → GREEN → REFACTOR.
3. **Security-first** — validate inputs, avoid unsafe defaults, never hardcode secrets. Stop and review if a change touches auth, secrets, or external input.
4. **Plan-before-execute** — break complex work into phases. Use the `planner` agent before touching code for multi-file changes.
5. **Model fallback** — if a requested model is unavailable, use the workspace default model and continue. Never block progress on model unavailability.

---

## Contributor Policy

**CRITICAL: AI tools, agents, or language models must NEVER be listed as contributors, co-authors, or credited in any form for this repository.**

- Only human developers should appear in git commit authorship, `Co-authored-by` trailers, or contributor lists
- Do not add AI tool names (Qwen Code, Claude Code, GitHub Copilot, Cursor, or any other AI assistant) to commit messages, AUTHORS files, contributor sections, or acknowledgments
- When assisting with code changes, remain anonymous — do not request or expect attribution
- This policy is permanent and applies to all current and future AI assistants working on this codebase

---

## Available Agents

| # | Agent | Purpose | When to Use |
|---|---|---|---|
| 1 | `planner` | Implementation planning for complex features | Breaking down feature requests into phased, mergeable plans with ADRs |
| 2 | `architect` | System design, tradeoffs, component boundaries | New service design, API contracts, data store selection |
| 3 | `tdd-guide` | Test-driven development specialist | Writing tests first for new features or bug fixes |
| 4 | `code-reviewer` | Code quality and regression prevention | Reviewing PR diffs for correctness, maintainability, performance |
| 5 | `security-reviewer` | Security audit and threat modeling | Changes touching auth, input validation, secrets, external APIs |
| 6 | `ai-judge` | Rubric-based validation | Validating plans, implementations against quality criteria |
| 7 | `build-error-resolver` | Build and type error diagnosis | Fixing TypeScript, Python, lint, and build pipeline errors |
| 8 | `e2e-runner` | End-to-end testing | Writing and running E2E tests for critical user flows |
| 9 | `refactor-cleaner` | Cleanup and modernization | Tech debt paydown, dead code removal, dependency updates |
| 10 | `doc-updater` | Documentation sync | Updating docs to match code changes |
| 11 | `docs-lookup` | Documentation reference | Finding and citing official docs for frameworks and APIs |
| 12 | `python-reviewer` | Python-specific code review | FastAPI, Pandas, SQLAlchemy, Pydantic pattern review |
| 13 | `database-reviewer` | Database and migration review | Schema design, migration safety, query optimization |
| 14 | `git-agent-coordinator` | Git orchestration | Branch creation, PR management, merge coordination |
| 15 | `ml-engineer` | ML/LLMOps specialist | RAG pipelines, model training, evals, deployment |
| 16 | `chrome-ext-developer` | WXT and Chrome extension work | Extension scaffolding, manifest config, permissions |
| 17 | `data-engineer` | ETL and data quality | Pipeline design, data validation, quality gates |
| 18 | `infra-as-code-specialist` | IaC and delivery pipelines | Terraform, CI/CD, deployment automation |
| 19 | `observability-telemetry` | Logs, metrics, traces, dashboards | Setting up monitoring, alerting, and observability |
| 20 | `multi-agent-project-manager` | Multi-workflow orchestration | Managing concurrent workflows, backlog, priority queue, never stops |
| 21 | `workflow-auditor` | Health checks and anomaly detection | Stuck workflow detection, quality gate trending, resource leaks |
| 22 | `reddit-researcher` | Reddit sentiment and experience mining | Real-world user experiences, production war stories, community consensus |
| 23 | `codebase-analyzer` | Codebase structure and complexity analysis | Understanding codebase architecture, dependency mapping |
| 24 | `codebase-learner` | Codebase learning specialist | Rapid onboarding to unfamiliar codebases |
| 25 | `code-quality-analyzer` | Static analysis and quality metrics | Code quality enforcement, standards checking |
| 26 | `test-debt-analyzer` | Test coverage and debt tracking | Identifying testing gaps, test debt prioritization |
| 27 | `security-debt-analyzer` | Security vulnerability tracking | Security debt identification, remediation planning |
| 28 | `performance-debt-analyzer` | Performance bottleneck analysis | Performance debt tracking, optimization opportunities |
| 29 | `dependency-debt-analyzer` | Dependency health monitoring | Outdated dependencies, upgrade path planning |
| 30 | `architecture-debt-analyzer` | Architectural debt detection | Design flaws, architectural drift detection |
| 31 | `process-debt-analyzer` | Workflow inefficiency detection | Process bottlenecks, workflow optimization |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noah-sheldon/ai-dev-kit](https://github.com/noah-sheldon/ai-dev-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
