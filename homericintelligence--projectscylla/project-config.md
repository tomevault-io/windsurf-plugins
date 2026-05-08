---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProjectScylla is an AI agent testing and optimization framework designed to measure, evaluate, and improve
the performance and cost-efficiency of agentic AI workflows. Named after the mythic trial from Homer's Odyssey,
Scylla represents the challenge of navigating trade-offs between capability gains and operational costs.

**Current Status**: Operational - active research with full evaluation infrastructure, running T0–T6 ablation
studies across 120 YAML subtests with published results and 75%+ test coverage enforced locally and in CI
(combined src/scylla/ + scripts/ floor; src/scylla/ unit coverage is also enforced at 75% in the CI unit step).

**Ecosystem Context**: Part of a 12-repository ecosystem:

| Repository | Role |
|------------|------|
| **AchaeanFleet** | Container images for the agent mesh — base images, Dockerfiles, Compose |
| **Myrmidons** | GitOps agent provisioning — agent definitions as code |
| **Odysseus** | CLI and core platform for agent lifecycle management |
| **ProjectArgus** | Observability — monitoring and metrics |
| **ProjectHephaestus** | Shared Python utilities and foundational tools |
| **ProjectHermes** | Webhook-to-NATS bridge — event ingestion |
| **ProjectKeystone** | DAG execution engine |
| **ProjectMnemosyne** | Skills marketplace — team knowledge sharing |
| **ProjectOdyssey** | Training and capability development for agents |
| **ProjectProteus** | CI/CD pipeline infrastructure |
| **ProjectScylla** | Testing, measurement, and optimization under constraints (this project) |
| **ProjectTelemachy** | Workflow engine |

## Critical Rules - Read First

### Never Push Directly to Main

**The `main` branch is protected. ALL changes MUST go through a pull request.**

**ABSOLUTELY PROHIBITED:**

```bash
git checkout main
git add <files>
git commit -m "changes"
git push origin main  # BLOCKED - Will be rejected by GitHub
```

**Why this is prohibited:**

- Bypasses code review and CI checks
- Can break production immediately
- Violates GitHub branch protection rules
- Makes it impossible to track changes properly

**CORRECT WORKFLOW (Always Use PRs):**

**Never use labels**

```bash
# 1. Create feature branch
git checkout -b <issue-number>-description

# 2. Make changes and commit
git add <files>
git commit -m "type(scope): description"

# 3. Push feature branch
git push -u origin <issue-number>-description

# 4. Create pull request
gh pr create \
  --title "[Type] Brief description" \
  --body "Closes #<issue-number>"

# 5. Enable auto-merge
gh pr merge --auto --rebase
```

**Emergency Situations:**

- Even for critical CI fixes, CREATE A PR
- Even for one-line changes, CREATE A PR
- Even if you're fixing your own mistake, CREATE A PR
- NO EXCEPTIONS - Always use the PR workflow

**See Also:**

- PR Best Practices: [PR Workflow](/.claude/shared/pr-workflow.md)

## Quick Links

### Core Guidelines

- [PR Workflow](/.claude/shared/pr-workflow.md)
- [GitHub Issue Workflow](/.claude/shared/github-issue-workflow.md)
- [Common Constraints](/.claude/shared/common-constraints.md)
- [Error Handling](/.claude/shared/error-handling.md)
- [Evaluation Guidelines](/.claude/shared/evaluation-guidelines.md)
- [Metrics Definitions](/.claude/shared/metrics-definitions.md)
- [Thinking Mode Configuration](/.claude/shared/thinking-mode.md) - Disabled by default

### Agent System

- [Agent Configurations](/.claude/agents/) - Evaluation-focused agents

## Working with Agents

This project uses a hierarchical agent system for all development work. **Always use agents** as the primary
method for completing tasks.

### Agent Hierarchy

Agent hierarchy is defined in `.claude/agents/` and `tests/claude-code/shared/agents/`:

- 6-level hierarchy (L0 Chief Evaluator → L5 Junior Engineers)
- Model assignments (Opus, Sonnet, Haiku)
- Specialized evaluation and benchmarking agents

### Key Agent Principles

1. **Always start with orchestrators** for new evaluation work
1. **All outputs** must be posted as comments on the GitHub issue
1. **Link all PRs** to issues using `gh pr create --issue <number>` or "Closes #123" in description
1. **Minimal changes only** - smallest change that solves the problem
1. **No scope creep** - focus only on issue requirements
1. **Reply to each review comment** with `Fixed - [brief description]`
1. **Delegate to skills** - Use "Use the X skill to..." pattern for automation

### Key Development Principles

1. KISS - *K*eep *I*t *S*imple *S*tupid -> Don't add complexity when a simpler solution works
1. YAGNI - *Y*ou *A*in't *G*onna *N*eed *I*t -> Don't add things until they are required
1. TDD - *T*est *D*riven *D*evelopment -> Write tests to drive the implementation
1. DRY - *D*on't *R*epeat *Y*ourself -> Don't duplicate functionality, data structures, or algorithms
1. SOLID - *S**O**L**I**D* ->
   . Single Responsibility
   . Open-Closed
   . Liskov Substitution
   . Interface Segregation
   . Dependency Inversion
1. Modularity - Develop independent modules through well defined interfaces
1. POLA - *P*rinciple *O*f *L*east *A*stonishment - Create intuitive and predictable interfaces


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HomericIntelligence/ProjectScylla](https://github.com/HomericIntelligence/ProjectScylla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
