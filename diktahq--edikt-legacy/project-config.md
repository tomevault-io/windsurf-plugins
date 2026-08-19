---
trigger: always_on
description: edikt is a governance layer for agentic engineering. It enforces your coding standards, persists your architectural decisions, and makes agent behavior reproducible across every session and every engineer.
---

# CLAUDE.md

## What This Project Is

edikt is a governance layer for agentic engineering. It enforces your coding standards, persists your architectural decisions, and makes agent behavior reproducible across every session and every engineer.

This project dogfoods itself: `.edikt/` governs edikt's own development.

## Architecture

All commands are `.md` files — no build step, no compiled code, no runtime dependencies. Rule templates live in `templates/rules/`. Configuration lives in `.edikt/config.yaml`.

**Repo structure:**
```
edikt/
├── commands/                 # 11 edikt slash commands
├── templates/
│   ├── rules/
│   │   ├── _registry.yaml   # maps rules to templates + metadata
│   │   ├── base/             # language-agnostic rules
│   │   ├── lang/             # language-specific rules
│   │   └── framework/        # framework-specific rules
│   ├── agents/               # agent templates
│   ├── sdlc/                 # PR templates, commit conventions
│   ├── CLAUDE.md.tmpl
│   ├── settings.json.tmpl
│   └── project-context.md.tmpl
├── test/                     # bash test harness
├── docs/
│   ├── architecture/
│   │   ├── decisions/        # ADRs
│   │   └── invariants/       # hard rules
│   ├── plans/
│   └── guides/
├── install.sh
└── README.md
```

## Key Invariants

- Commands are `.md` files — no compiled code, no build step
- Installation is copy files — no npm, no dependencies
- Rule templates are single `.md` per topic (not folders with individual rules)
- Three-tier rules: base (language-agnostic), lang, framework
- Claude Code only for execution reliability

## Before Implementing

1. Read ADR-001 in `docs/architecture/decisions/`
2. Read INV-001 in `docs/architecture/invariants/`
3. Read the implementation plan in `docs/plans/PLAN-edikt-v1.md`
4. Read `.edikt/project-context.md` for project identity

## Testing

Run `./test/run.sh` to validate templates, registry, and install.

## Commit Convention

```
{type}({scope}): {description}
```

Types: feat | fix | refactor | test | docs | chore

[edikt:start]: # managed by edikt — do not edit this block manually
## edikt

### Project
edikt is the governance layer for agentic engineering. It enforces coding standards, persists architectural decisions, and makes agent behavior reproducible across every session and every engineer.

### Before Writing Code
1. Read `docs/project-context.md` for project context
2. Rules are enforced automatically via `.claude/rules/`
3. If a plan is active, read it in `docs/plans/` — check progress table for current state
4. If a spec exists, read it in `docs/product/specs/` — the spec and its artifacts are the engineering blueprint
5. All paths are configurable in `.edikt/config.yaml` under `paths:`

### Build & Test Commands
```
# Test
./test/run.sh

# Install (global)
curl -fsSL https://raw.githubusercontent.com/diktahq/edikt/main/install.sh | bash
```

### edikt Commands

Match the user's intent, not their exact words. These are representative examples — if the meaning is the same, run the command.

| Intent | Examples | Run |
|--------|----------|-----|
| Project status / what's next | "what's our status", "where are we", "what's next", "project status", "next steps" | `/edikt:status` |
| Load project context | "load context", "remind yourself", "what's this project", "give me context" | `/edikt:context` |
| Create an execution plan | "create a plan", "make a plan", "let's plan this", "plan for X", "plan this ticket", "help me plan", "how should we approach X", "plan [ticket ID]", "continue the plan", "re-plan phase 3", "let's create a plan to fix X", "plan to fix these issues", "plan these changes", "plan this work" | `/edikt:sdlc:plan` |
| Capture an architecture decision | "save this decision", "record this", "capture that", "write an ADR", "document this decision" | `/edikt:adr:new` |
| Add a hard constraint | "add an invariant", "that's a hard rule", "never do X", "this must always be true" | `/edikt:invariant:new` |
| Write a PRD | "write a PRD", "document this feature", "requirements for X", "product requirements" | `/edikt:sdlc:prd` |
| Write a technical spec | "write a spec", "technical spec for X", "spec this out", "design doc for X" | `/edikt:sdlc:spec` |
| Generate spec artifacts | "generate artifacts", "create the data model", "generate the contracts", "build the artifacts" | `/edikt:sdlc:artifacts` |
| Check implementation drift | "check drift", "did we build what we decided", "verify the implementation", "are we on track with the spec" | `/edikt:sdlc:drift` |
| Compile governance | "compile governance", "update directives", "update the rules" | `/edikt:gov:compile` |
| Review governance quality | "review governance", "are our ADRs well written", "check governance quality" | `/edikt:gov:review` |
| Review implementation | "review what we built", "post-implementation review", "review this code" | `/edikt:sdlc:review` |
| Security audit | "run a security audit", "check for vulnerabilities", "security check" | `/edikt:sdlc:audit` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diktahq/edikt-legacy](https://github.com/diktahq/edikt-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
