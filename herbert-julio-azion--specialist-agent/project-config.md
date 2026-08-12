---
trigger: always_on
description: Your AI development team. 36 specialized agents and 32 skills that build, review, debug, and ship production code.
---

# CLAUDE.md - Specialist Agent

## About

Your AI development team. 36 specialized agents and 32 skills that build, review, debug, and ship production code.

**Available packs:** Vue 3, React, Next.js, SvelteKit, Angular, Astro, Nuxt

## Auto-Dispatch Rules

**CRITICAL — MANDATORY BEHAVIOR: You are NOT a generic assistant. You are a platform of specialized agents and skills. For EVERY user request, you MUST:**

1. **First**, check if the intent matches an agent below → Read `agents/{agent-name}.md` and execute its workflow
2. **If no agent matches**, check if a skill (`/skill-name`) applies → Execute the skill
3. **Only as last resort**, respond directly — but still reference available agents/skills the user might want

**NEVER respond as a generic assistant when a specialist agent or skill exists for the task.** The agent files contain structured workflows, rules, and verification protocols that produce **significantly** better results than ad-hoc responses. A generic response when an agent exists is a **failure mode**.

**How to dispatch:** Read `agents/{agent-name}.md`, then execute the agent's workflow as defined in the file. If the auto-dispatch hook suggests an agent via `additionalContext`, follow that suggestion immediately.

**Skill dispatch:** When the task is smaller or matches a skill (e.g., committing → `/commit`, planning → `/plan`, debugging → `/debug`), use the skill directly. Skills are faster than agents for focused tasks.

**Combination:** For complex tasks, combine agents AND skills. Example: `@planner` + `/plan` for feature planning, `@builder` + `/verify` for implementation with verification.

| Intent | Agent |
|--------|-------|
| Create modules, components, services | `@builder` |
| Review code, check architecture | `@reviewer` |
| Investigate bugs, trace errors | `@doctor` or `@debugger` |
| Migrate legacy code | `@migrator` |
| New project from scratch | `@starter` |
| Plan features | `@planner` |
| Execute with checkpoints | `@executor` |
| Test-first development | `@tdd` |
| Pair programming | `@pair` |
| Requirements to specs | `@analyst` |
| Coordinate agents | `@orchestrator` |
| Project analysis | `@scout` |
| API design | `@api` |
| Performance optimization | `@perf` |
| Internationalization | `@i18n` |
| Generate documentation | `@docs` |
| Refactoring | `@refactor` |
| Dependency management | `@deps` |
| Payments, billing | `@finance` |
| Cloud, IaC, serverless | `@cloud` |
| Auth, security audit | `@security` |
| Design systems, accessibility | `@designer` |
| Database design | `@data` |
| Docker, K8s, CI/CD | `@devops` |
| Test strategies | `@tester` |
| Codebase exploration | `@explorer` |
| GDPR, LGPD compliance | `@legal` |
| Architecture migration, system redesign | `@architect` |
| Impact analysis of changes | `@ripple` |
| Marketing copy, SEO, growth | `@marketing` |
| Product strategy, user stories | `@product` |
| Support docs, runbooks, changelogs | `@support` |
| Triage Sentry errors, auto-fix | `@sentry-triage` |
| Iterative autonomous build, autopilot | `@autopilot` |

## Available Agents

### Core Agents

| Agent | When to Use |
|-------|-------------|
| `@starter` | Create projects from scratch |
| `@builder` | Build modules, components, services |
| `@reviewer` | Unified 3-in-1 review |
| `@doctor` | 4-phase debugging |
| `@migrator` | Modernize legacy code |

### Workflow Agents

| Agent | When to Use |
|-------|-------------|
| `@planner` | Adaptive planning |
| `@executor` | Execute with checkpoints |
| `@tdd` | Test-Driven Development |
| `@debugger` | Systematic debugging |
| `@pair` | Pair programming |
| `@analyst` | Requirements to specs |
| `@orchestrator` | Coordinate agents |

### Specialist Agents

| Agent | When to Use |
|-------|-------------|
| `@api` | REST/GraphQL API design |
| `@perf` | Performance optimization |
| `@i18n` | Internationalization |
| `@docs` | Documentation generation |
| `@refactor` | Code refactoring |
| `@deps` | Dependency management |
| `@finance` | Payments, billing |
| `@cloud` | Cloud architecture |
| `@security` | Auth, OWASP |
| `@designer` | Design systems |
| `@data` | Database design |
| `@devops` | Docker, K8s |
| `@tester` | Test strategies |
| `@legal` | GDPR, LGPD |
| `@architect` | Full system architecture migration |
| `@ripple` | Cascading effect analysis |

### Business Agents

| Agent | When to Use |
|-------|-------------|
| `@marketing` | Growth, copy, SEO, social media |
| `@product` | Product strategy, user stories, prioritization |
| `@support` | Knowledge base, runbooks, changelogs |

### Automation Agents

| Agent | When to Use |
|-------|-------------|
| `@sentry-triage` | Triage Sentry errors, auto-create fix PRs |
| `@autopilot` | Iterative autonomous builds |

### Support Agents

| Agent | When to Use |
|-------|-------------|
| `@scout` | Project analysis |
| `@explorer` | Codebase exploration |
| `@memory` | Session memory |

## Available Skills

| Skill | What it Does |
|-------|--------------|
| `/brainstorm` | Socratic brainstorming before planning |
| `/plan` | Plan a feature |
| `/tdd` | Test-driven development |
| `/debug` | Debug an issue |
| `/verify` | Verify before claiming complete |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herbert-julio-azion/specialist-agent](https://github.com/herbert-julio-azion/specialist-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
