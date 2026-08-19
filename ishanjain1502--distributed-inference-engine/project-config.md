---
trigger: always_on
description: Handles edge cases like missing values, type mismatches,
---

# AGENTS.md — Anchor

> The production-grade template for AI coding agent instructions. Adapt sections to fit project scope. Keep your agents grounded.
>
> **This file governs itself.** Agents working on AGENTS.md must follow all rules herein — the decision ladder, commit conventions, tradeoff comments, quality gates, and output discipline. No section should be added that future agents would need to explain away as inapplicable.

---

## Section Index

| Domain | Sections | Covers |
|--------|----------|--------|
| **Core** | [1](#1-core-principles)–[9](#9-linting--type-checking) | Principles, commit protocol, shell rules, code style, project structure, TODO.md, Docker, testing, linting |
| **Architecture** | [10](#10-error-handling-patterns)–[20](#20-external-integrations) | Error handling, config, API design, security, logging, git workflow, docs, deps, performance, build/deploy, external integrations |
| **AI Agent Guidance** | [21](#21-ai-agent-instruction-guidance)–[28](#28-default-tech-stack-playbook) | Instruction guidance, multi-agent patterns, verification gates, failure modes, gotchas, getting help, code quality standards, tech stack playbook |
| **Production Ops** | [29](#29-operational-patterns)–[32](#32-docker-support) | Operational patterns (circuit breaker, DLQ, middleware, cache), health endpoints, production security, Docker & Kubernetes |
| **Process** | [33](#33-pr--change-size-standards)–[41](#41-observability-standards) | PR size standards, AI anti-pattern detection, PR template, NEVER list, pre-commit hooks, CI/CD pipelines, semantic versioning, code coverage, observability |
| **Infrastructure** | [42](#42-infrastructure-as-code)–[44](#44-secrets-management) | IaC (Terraform), database backup & recovery, secrets management |
| **Testing Excellence** | [45](#45-flaky-test-management)–[49](#49-chaos-engineering) | Flaky test management, mutation testing, performance benchmarks, contract testing (Pact), chaos engineering |
| **Philosophy** | [50](#50-intentional-minimalism--the-simplicity-first-architecture)–[51](#51-instruction-architecture--context-economy--self-improvement) | Intentional minimalism (decision ladder), instruction architecture (lazy loading, self-maintenance, context budgets) |
| **Enforcement** | [52](#52-rule-enforcement-architecture--from-advisory-to-deterministic) | Two-layer enforcement model, prose-to-hook compilation, evidence-first methodology, CI compliance gates |
| **Project Types** | [53](#53-project-type-patterns) | Language-agnostic and project-type-specific patterns for mobile, embedded, data pipeline, CLI, static site, plus coverage scope |

## Quick-Navigation Cheatsheet

**Agent — how to find what you need without reading the entire document:**

| I need to... | Go to Section |
|-------------|---------------|
| Commit code | [2](#2-commit-protocol), [15](#15-git-workflow), [9](#9-linting--type-checking) |
| Create a PR | [33](#33-pr--change-size-standards), [35](#35-pr-description-format--template), [34](#34-ai-code-quality--anti-pattern-detection) |
| Review a PR | [33.2](#332-when-changes-exceed-the-limit), [50.5](#505-over-engineering-review-vocabulary), [36](#36-explicit-prohibitions--the-never-list) |
| Write tests | [8](#8-testing-requirements), [40](#40-code-coverage-enforcement), [45](#45-flaky-test-management) |
| Fix a security issue | [13](#13-security-best-practices), [27.3](#273-security-constraints--mandatory), [31](#31-production-security-patterns), [44](#44-secrets-management) |
| Set up a new project | [5](#5-project-structure-conventions), [37](#37-pre-commit-hook-standards), [38](#38-cicd-pipeline-standards), STARTUP.md |
| Deploy to production | [32](#32-docker-support), [38.3](#383-deployment-pipeline--githubworkflowsdeployyml), [30](#30-health-endpoint-specification) |
| Handle a flaky test | [45](#45-flaky-test-management) — quarantine, don't delete, fix within 7 days |
| Debug a slow endpoint | [18](#18-performance-considerations), [41.6](#416-service-level-objectives-slos), [47](#47-performance-benchmark-testing) |
| Add a dependency | [17](#17-dependency-management), [50.1](#501-the-decision-ladder--stop-at-the-first-rung-that-holds) (decision ladder rung 4) |
| Refuse a dangerous request | [2](#2-commit-protocol) (Never Go Rogue + Never Spend Money), [36.4](#364-financial-never) — cite these sections |
| Decide if code is too simple to test | [50.7](#507-tests-are-not-bloat) — one runnable check per non-trivial function |
| Rollback a bad deployment | Rollback is always `git revert` first, then fix forward. Never fix on a broken deploy — roll back, then debug. |
| Handle a merge conflict | Resolve by choosing the more recent change for logic, the clearer documentation for comments. Run full test suite after resolution. |
| Rotate secrets | [44.4](#444-secret-rotation-pattern) — dual-key window pattern, deploy new key, promote, remove old |
| Working on a non-web project (mobile, embedded, data pipeline, CLI, static site) | [53](#53-project-type-patterns) — language-agnostic and project-type-specific patterns for 5 common non-web project types |
| Want to know what this template covers vs doesn't | [53.7](#537-template-coverage-and-out-of-scope) — explicit list of well-covered, partial, and out-of-scope project types |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ishanjain1502/distributed-inference-engine](https://github.com/ishanjain1502/distributed-inference-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
