---
trigger: always_on
description: Teamwork is an agent-native development template that structures AI-human collaboration through defined roles (Custom Agents), workflows (Skills), and conventions.
---

# Copilot Instructions — Teamwork

Teamwork is an agent-native development template that structures AI-human collaboration through defined roles (Custom Agents), workflows (Skills), and conventions.

## Before You Start

0. **Read project context.** Start every session by reading `MEMORY.md` for current project state, recent decisions, and active context.

1. **Select your agent.** Choose the appropriate Custom Agent from `.github/agents/` — each agent has a defined persona, tool restrictions, and boundaries:

   **Core agents:**
   - `@planner` — Break goals into tasks. Never write code.
   - `@architect` — Design systems, write ADRs. Never write code.
   - `@coder` — Implement tasks, write tests, open PRs.
   - `@tester` — Write adversarial tests. Never modify production code.
   - `@reviewer` — Review PRs for quality. Never modify code.
   - `@security-auditor` — Audit for vulnerabilities. Never modify code.
   - `@documenter` — Keep documentation accurate and current.
   - `@orchestrator` — Coordinate workflows, dispatch roles. Never write code.

   **Extended agents:**
   - `@triager` — Triage and classify incoming issues.
   - `@devops` — CI/CD pipelines, infrastructure, deployments.
   - `@dependency-manager` — Update and audit third-party dependencies.
   - `@refactorer` — Restructure code without changing behavior.
   - `@lint-agent` — Fix code style and formatting issues.
   - `@api-agent` — Design and build API endpoints.
   - `@dba-agent` — Database schema, migrations, query optimization.

   **If no agent is specified:** Use `docs/role-selector.md` to determine the right one. Quick defaults: implementation → Coder, planning → Planner, code review → Reviewer, multi-role → Planner (break it down first).

2. **Read the conventions.** Review `docs/conventions.md` for coding standards, branch naming, commit format, and PR requirements.
3. **Understand the architecture.** Check `docs/architecture.md` for prior design decisions (ADRs) before proposing structural changes.
4. **Learn the vocabulary.** Use terminology as defined in `docs/glossary.md`.
5. **Invoke a workflow skill.** For multi-step tasks, use `/skill-name` or check `.github/skills/` for structured workflows:
   - `/feature-workflow` — Adding new functionality
   - `/bugfix-workflow` — Diagnosing and fixing bugs
   - `/refactor-workflow` — Restructuring existing code
   - `/hotfix-workflow` — Urgent production fixes
   - `/security-response` — Responding to security vulnerabilities
   - `/dependency-update` — Updating third-party dependencies
   - `/documentation-workflow` — Standalone documentation updates
   - `/spike-workflow` — Research or technical investigation
   - `/release-workflow` — Preparing and publishing releases
   - `/rollback-workflow` — Rolling back failed deployments
   - `/setup-teamwork` — Fill in all CUSTOMIZE placeholders by analyzing the repo

## Agent & Skill Usage

When a user request matches what a custom agent is designed to do, **always dispatch that agent** via the `task` tool instead of doing the work yourself. Treat custom agents the same way skills are treated — match the request to the agent's description and invoke it automatically.

**Agent dispatch rules:**
- Implementation or coding work → dispatch `@coder`
- Design decisions or architecture review → dispatch `@architect`
- Writing or improving tests → dispatch `@tester`
- Code review → dispatch `@reviewer`
- Security concerns or audits → dispatch `@security-auditor`
- Planning or breaking down work → dispatch `@planner`
- Documentation updates → dispatch `@documenter`
- CI/CD or deployment tasks → dispatch `@devops`
- Code refactoring → dispatch `@refactorer`
- Database schema or queries → dispatch `@dba-agent`
- Dependency updates or audits → dispatch `@dependency-manager`
- Issue triage → dispatch `@triager`
- API design or endpoints → dispatch `@api-agent`
- Code style or formatting fixes → dispatch `@lint-agent`

**Skills** (in `.github/skills/`) are invoked automatically when the request matches a workflow pattern. **Agents** (in `.github/agents/`) should be dispatched with the same automatic behavior via the `task` tool.

**Why this matters:** Without explicit dispatch, Copilot will attempt everything itself rather than delegating to specialized agents — defeating the purpose of role-based architecture. Each agent has domain-specific expertise, boundaries, and quality standards defined in its `.agent.md` file.

## Key Rules

- **Minimal changes.** Change only what is necessary. Do not refactor unrelated code.
- **Test before submitting.** Run all relevant tests and verify they pass before opening a PR.
- **Conventional commits.** Format: `type(scope): description` (e.g., `feat(auth): add token refresh`).
- **One task per PR.** Keep pull requests focused on a single task or change.
- **Respect agent boundaries.** Each agent's `.agent.md` file defines ✅ Always / ⚠️ Ask first / 🚫 Never rules. Follow them.
- **Keep scope small.** Target ~300 lines changed and ~10 files maximum per task.

## When to Escalate

Stop and ask the human when:

- Requirements are ambiguous or contradictory
- A change would affect architecture or public APIs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [POps-Rox/terraform-az-overlays-containerinstance](https://github.com/POps-Rox/terraform-az-overlays-containerinstance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
