---
trigger: always_on
description: - **1 sub-agent** — default. Good for one focused task (exploration, data fetch, implementation).
---

# Engineering Standards

## Sub-agent Discipline

- **1 sub-agent** — default. Good for one focused task (exploration, data fetch, implementation).
- **2 sub-agents** — great. Use for two genuinely independent parallel tasks with no shared state.
- **3 sub-agents** — only when 3 tasks are clearly independent, time-critical, and cannot share context. Justify before spawning.
- **Never spawn 4+** in a single turn.

Use `haiku` for read-only exploration or data fetch. Use `sonnet` for code writing or reasoning.

## Tool Preferences
- **LSP first**: Use LSP (go-to-definition, find-references, diagnostics) for code navigation — grep only when LSP not applicable
- **context7 for docs**: Always fetch current docs via context7 for any library/framework/SDK/API — never rely on training data alone
- **Notifications**: Use PushNotification when waiting >30s on external process, CI, or user input
- **Parallel work**: Use `superpowers:using-git-worktrees` for parallel feature branches
- **Never read `.env` or `.envrc`**: May contain production secrets — never read, never echo, never log
- **After git push**: Hook surfaces open PR comments automatically — for each: if valid issue (bug/missing test/security hole), fix the code + reply via `gh pr comment` explaining what changed; if not actionable (style preference/opinion/already done), reply explaining why. Never leave PR comments unanswered.

## Pipeline & Skills (always use before acting)

Before writing code, designing architecture, reviewing security, or running quality gates — invoke the matching skill. Never free-form tasks that have a defined skill.

| Task | Skill | Trigger phrases |
|------|-------|-----------------|
| New feature / epic / large task | `/multi-agent-coding-pipeline` | "build", "create", "new feature", "epic", "implement X from scratch" |
| Bug investigation and fix | `/bug-fix` | "bug", "fix", "broken", "not working", "wrong behavior", "unexpected", "crash", "regression", "debug", "fails with" |
| Single task, small feature | `/task-coding-pipeline` | "small change", "quick task", "add X to existing" |
| Architecture design | `/architecture` | "architect", "design the system", "how should we structure", "system design" |
| Requirements analysis (no code, no plan) | `/analysis` | "analyze requirements", "assess", "evaluate context", "investigate requirements", "what should we build", "what do we need" |
| Execution plan (no implementation) | `/planning` | "plan", "planning", "make a plan", "create execution plan", "break down into tasks", "roadmap", "how would we approach" |
| Security audit | `/security-review` | "security", "audit", "vulnerability", "OWASP", "pen test", "check for issues", "prompt injection", "LLM security", "LLM01", "AI security", "GenAI risk" |
| Quality gates / CI check | `/quality-gate` | "quality gate", "run gates", "CI check", "lint", "coverage", "run tests" |
| PR review or post-push comments | `/pr-review` | "review PR", "check PR", "PR comments", "code review", "review this diff" |
| Business rules mapping | `/business-analysis` | "business rules", "business logic", "domain rules", "what does the business require" |
| Technical contract mapping | `/technical-analysis` | "technical contract", "interface design", "API contract", "map the interfaces" |
| Cut a release | `/release-management` | "release", "cut a release", "ship", "version", "tag", "changelog" |
| Write a DB migration | `/database-migration` | "migration", "db migration", "schema change", "add column", "alter table" |
| Add logging / metrics / tracing | `/observability` | "logging", "metrics", "tracing", "observability", "add logs", "instrument" |
| Performance investigation | `/performance-profiling` | "performance", "slow", "profiling", "optimize", "latency", "throughput" |
| Build integration adapter | `/rote` | "connect to", "integrate with", "add adapter", "add integration", "rote", "new connector" |
| TDD implementation | `/superpowers:test-driven-development` | direct code ask outside a pipeline — "write this function", "implement this method", "add this helper", small focused coding not warranting a full pipeline |
| Stress-test a plan/design | `/grill-me` | "grill me", "challenge this", "stress-test", "poke holes", "pick this apart" |
| Architectural health review | `/improve-codebase-architecture` | "improve architecture", "zoom out", "architectural review", "find coupling", "codebase health" |
| End-of-session handoff doc | `/handoff` | "handoff", "wrap up", "end session", "save context", "compact this session" |
| Create a new skill | `/write-a-skill` | "write a skill", "create skill", "add skill", "new skill" |

**Rule**: If the user's message contains any trigger phrase above — or the intent clearly matches a row — invoke the skill first. Do not start writing code or analysis until the skill has been loaded. A task that "feels simple" is not an exception.

**Agents are loaded by pipeline skills** — never load `agents/*.md` files manually unless a pipeline skill instructs it.

## Coding Discipline (12 rules — non-negotiable)

1. **Think before coding**: State assumptions, ask questions, stop when confused. Never guess.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuisFelipeMoro/Claude-devkit](https://github.com/LuisFelipeMoro/Claude-devkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
