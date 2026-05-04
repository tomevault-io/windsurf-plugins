---
trigger: always_on
description: > Stack: [e.g., Next.js 14 · TypeScript · PostgreSQL · Prisma · Railway]
---

# [Project Name] — Claude Instructions

> Stack: [e.g., Next.js 14 · TypeScript · PostgreSQL · Prisma · Railway]
> Last updated: [YYYY-MM-DD]

## Project Context

[2–3 sentences: what this product does, who it serves, and the core problem it solves.]

**Tech stack summary**: [Frontend] · [Backend] · [Database] · [Hosting]

---

## Agents Available

**Mandatory delegation — this is not optional.** Every task that falls within a specialist's domain MUST be routed to that agent. Do not implement code, design schemas, write docs, or configure pipelines yourself — delegate. Only handle directly: project-level questions, routing decisions, and tasks explicitly outside all specialist domains.

| Agent | Role | Invoke when... |
|-------|------|----------------|
| `planner` | Backlog & architecture | "What's next?", sprint planning, feature decomposition, tech decisions, ADRs, new feature design before implementation |
| `builder` | All application code | Frontend components, backend endpoints, database schemas/migrations, React Native screens — specify domain in request |
| `designer` | UX & content | User flows, wireframes, component specs, accessibility, landing copy, SEO, brand voice, marketing content |
| `quality` | Testing & documentation | E2E tests, test strategy, coverage gaps, user guide updates, post-feature documentation |
| `infra` | Infrastructure & pipelines | CI/CD workflows, deployments, Dockerfiles, docker-compose, container configuration |

---

## Critical Rules

These apply to all agents at all times. No exceptions without explicit human instruction.

1. **PRD.md requires explicit human approval to modify.** Do not edit it unless the human has clearly instructed you to do so in the current conversation. Read it to understand requirements.
2. **TODO.md is the living backlog.** Agents may add items, mark items complete, and move items to "Completed". Preserve section order and existing item priority — do not reorder items within a section unless explicitly asked to reprioritize.
3. **All commits use Conventional Commits format** (see Git Conventions below).
4. **Update the relevant `docs/` file** after every significant change before marking a task complete.
5. **Run tests before marking any implementation task complete.**
6. **Never hardcode secrets, credentials, or environment-specific values** in source code.
7. **Consult `docs/technical/DECISIONS.md`** before proposing changes that may conflict with prior architectural decisions.
8. **Always delegate to the right specialist.** If a task touches application code (frontend, mobile, backend, or database), design/UX/content, testing/documentation, or infrastructure — invoke the appropriate agent (`builder`, `designer`, `quality`, or `infra`) immediately. Do not implement it yourself. The delegation table above is binding, not advisory.
9. **Commit your own changes; never push.** After completing your work, create a local commit (Conventional Commits format). Do not `git push`. The orchestrator is responsible for pushing the branch and opening the PR.
10. **When invoking `builder`, specify the domain in your request** (e.g. "frontend task — add dark mode toggle" or "database task — add index on orders table"). The builder reads the corresponding skill before starting work.

---

## Slash Commands

Use these commands to trigger common multi-step workflows:

| Command | What it does |
|---------|--------------|
| `/orchestrate <task>` | Full multi-agent task execution — decompose, plan, branch, execute in waves |
| `/status` | Render a live project health card (tasks, commits, open PRs, blockers) |
| `/start` | Run project onboarding from `START_HERE.md` |
| `/sync-template` | Pull latest agent definitions and templates from upstream |

---

## MCP Servers

Project MCP servers are declared in `.mcp.json` (committed to the repo — shared by the whole team). No extra credentials required — both servers are unauthenticated.

| Server | Purpose | Agents that use it |
|--------|---------|-------------------|
| `sequential-thinking` | Structured multi-step reasoning scratchpad | `planner` |
| `context7` | Live, version-accurate library documentation | `builder`, `infra` |

**GitHub integration** — use the `gh` CLI (already authenticated via `gh auth login`). All agents with `Bash` access can run `gh` commands directly. No token configuration needed.

---

## File-Scoped Rules

Rules in `.claude/rules/` inject context automatically based on the file being edited:

| Rule file | Applied to | Key standards |
|-----------|-----------|---------------|
| `typescript.md` | `*.ts`, `*.tsx` | No `any`, no `!` assertions, no `console.log`, explicit return types |
| `migrations.md` | `*.sql`, `migrations/**` | Reversible migrations, naming convention, no destructive ops without guards |
| `tests.md` | `*.spec.ts`, `*.test.ts`, `tests/**` | Page Object Model, `data-testid` selectors, no `test.only`, 80% coverage |

---

## Project Structure

```
src/                    # Application source code
  app/                  # [e.g., Next.js App Router pages]
  components/           # Shared UI components
  lib/                  # Utilities, helpers, shared logic
tests/
  e2e/                  # Playwright E2E tests (*.spec.ts)
docs/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josipjelic/orchestrated-project-template](https://github.com/josipjelic/orchestrated-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
