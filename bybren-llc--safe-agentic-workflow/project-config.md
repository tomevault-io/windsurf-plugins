---
trigger: always_on
description: > **Philosophy**: "Search First, Reuse Always, Create Only When Necessary"
---

# SAFe Agent Team Quick Reference

> **Philosophy**: "Search First, Reuse Always, Create Only When Necessary"
>
> Pattern discovery is MANDATORY before implementation.
>
> **Team Culture**: "We work as a round table team that has 4 pillars of SAFe inscribed on that round table. It means something."

## Documentation

**Workflow SOPs:**

- [Agent Workflow SOP v1.4](./docs/sop/AGENT_WORKFLOW_SOP.md) - vNext contract, Exit States, Role Collapsing ({{TICKET_PREFIX}}-497/499)
- [Agent Configuration SOP](./docs/sop/AGENT_CONFIGURATION_SOP.md) - Tool restrictions, model selection
- [ARCHitect-in-CLI Role](./docs/workflow/ARCHITECT_IN_CLI_ROLE.md) - Primary orchestrator definition

**CI/CD Documentation:**

- [CI/CD Pipeline Guide](./docs/ci-cd/CI-CD-Pipeline-Guide.md) - Pipeline implementation guide

**Database SOPs:**

- [RLS Migration SOP](./docs/database/RLS_DATABASE_MIGRATION_SOP.md) - MANDATORY for Data Engineer

**Project Standards:**

- [Harness Whitepaper](./docs/whitepapers/CLAUDE-CODE-HARNESS-MODERNIZATION-{{TICKET_PREFIX}}-444.md) - Complete harness architecture
- [Agent Perspective](./docs/whitepapers/CLAUDE-CODE-HARNESS-AGENT-PERSPECTIVE.md) - Why the harness works
- [SAFe Methodology](https://github.com/{{GITHUB_ORG}}/{{PROJECT_REPO}}) - This repository

## When to Use Which Agent

| Agent Role                           | Use Case                                                                                          | Success Criteria                                            | Primary Tools                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------- |
| **TDM** (Technical Delivery Manager) | Reactive blocker resolution, Linear updates, evidence tracking (NOT orchestration - see v1.3 SOP) | Blockers resolved, evidence attached, Linear updated        | Linear, Confluence                  |
| **BSA** (Business Systems Analyst)   | Requirements decomposition, acceptance criteria, testing strategy                                 | Clear user stories, testable ACs, QA plan defined           | Linear, Confluence, Markdown        |
| **System Architect**                 | Pattern validation, Stage 1 PR review, migration approval, architectural decisions                | ADR created, PR technical review complete, no conflicts     | Read, Grep, ADR templates           |
| **FE Developer**                     | UI components, client-side logic, user interactions                                               | Lint and build passes                                       | Read, Write, Edit, Bash             |
| **BE Developer**                     | API routes, server logic, RLS enforcement                                                         | Integration tests pass                                      | Read, Write, Edit, Bash             |
| **DE** (Data Engineer)               | Schema changes, migrations, database architecture                                                 | Migration applied, RLS maintained                           | Prisma, SQL, migration tools        |
| **TW** (Technical Writer)            | Documentation, guides, technical content                                                          | Markdown lint passes                                        | Read, Write, Edit, Grep, Glob, Bash |
| **DPE** (Data Provisioning Engineer) | Test data, database access, data validation                                                       | Test data available, DB accessible                          | SQL, Prisma Studio, scripts         |
| **QAS** (Quality Assurance)          | **GATE OWNER**: Execute testing, validate ACs, iteration authority, evidence to Linear            | All ACs verified, evidence posted, Exit: "Approved for RTE" | Playwright, Jest, Linear MCP        |
| **SecEng** (Security Engineer)       | Security validation, RLS checks, vulnerability assessment (Independence Gate - not collapsible)   | Security audit passed, RLS enforced                         | RLS scripts, security tools         |
| **RTE** (Release Train Engineer)     | **PR SHEPHERD**: PR creation, CI/CD monitoring (NO code, NO merge) - Exit: "Ready for HITL"       | PR created, CI green, Exit: "Ready for HITL Review"         | Git, GitHub CLI, CI tools           |

## Auto-Loaded Skills

Skills are loaded progressively—metadata at startup, full content when context triggers.

| Skill                    | Trigger                | Purpose                                     |
| ------------------------ | ---------------------- | ------------------------------------------- |
| `safe-workflow`          | Commits, branches, PRs | SAFe format, rebase-first workflow          |
| `pattern-discovery`      | Before writing code    | Pattern-first development (MANDATORY)       |
| `rls-patterns`           | Database operations    | RLS context helpers (withUserContext, etc.) |
| `frontend-patterns`      | UI work                | Clerk, shadcn, Next.js patterns             |
| `api-patterns`           | API route creation     | Route structure, error handling             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bybren-llc/safe-agentic-workflow](https://github.com/bybren-llc/safe-agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
