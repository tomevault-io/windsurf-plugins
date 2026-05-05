---
trigger: always_on
description: Quick-reference for AI coding agents and human contributors working in this repo.
---

# AGENTS.md

Quick-reference for AI coding agents and human contributors working in this repo.

## What this repo is

Agentic Delivery OS is a spec-driven software delivery system: a team of AI agents and commands that turn a ticket into a reviewed, tested PR through a deterministic 10-phase workflow.

The agents and their prompt definitions (`.opencode/agent/*.md`, `.opencode/command/*.md`) **are the product**. A degraded prompt degrades everything downstream — treat them with the same rigor as production code. The delivery process is used to deliver improvements to itself.

> **New to ADOS?** See [doc/guides/onboarding-existing-project.md](doc/guides/onboarding-existing-project.md) or run `/bootstrap` to get started.

## Delivery process

Every change flows through 10 phases. `@pm` orchestrates; phases are gated but can be reopened when gaps are discovered.

| Phase | Agent | What happens |
|-------|-------|--------------|
| 1. clarify_scope | `@pm` | Read ticket via MCP, cross-check against system spec (`doc/spec/**`), STOP if questions |
| 2. specification | `@spec-writer` | Create `chg-<ref>-spec.md` (problem, goals, AC) |
| 3. test_planning | `@test-plan-writer` | Create `chg-<ref>-test-plan.md` (traceable to AC) |
| 4. delivery_planning | `@plan-writer` | Create `chg-<ref>-plan.md` (phased tasks) |
| 5. delivery | `@coder` | Execute plan phases, commit per phase |
| 6. system_spec_update | `@doc-syncer` | Reconcile `doc/spec/**` with implementation |
| 7. review_fix | `@reviewer` | Audit vs spec/plan; if FAIL → `@coder` remediates → re-review |
| 8. quality_gates | `@runner` | Build/test/lint; if failures → `@fixer` → re-run |
| 9. dod_check | `@pm` | Verify all AC met, all plan tasks done |
| 10. pr_creation | `@pr-manager` | Create PR, assign to human, STOP |

Detail: [doc/guides/change-lifecycle.md](doc/guides/change-lifecycle.md)

## Agent team

### Orchestration
- `pm` — orchestrate changes; manage tickets via MCP; never implements code
- `architect` — architecture decisions and decision record authoring (ADR/PDR/TDR/BDR/ODR)

### Onboarding
- `bootstrapper` — automate ADOS adoption for existing projects

### Artifact creation
- `spec-writer` — author change specifications
- `plan-writer` — author implementation plans
- `test-plan-writer` — author test plans with traceable coverage

### Implementation
- `coder` — execute plan phases; delegates to `@designer`, `@architect`, `@committer`, `@runner`
- `designer` — visual design and UI implementation
- `editor` — content rewrites and translations

### Verification
- `review-feedback-applier` — classify and apply accepted review feedback from PR/MR
- `reviewer` — review changes against spec, plan, code quality heuristics, and repo rules (local + remote modes)
- `fixer` — reproduce failures and apply targeted fixes
- `runner` — execute commands, capture logs (subagent)

### Documentation & release
- `doc-syncer` — reconcile system docs with completed changes
- `committer` — create one Conventional Commit
- `pr-manager` — create/update PR/MR; enrich with ticket context via MCP

### Specialized
- `external-researcher` — research via MCP (context7, deepwiki, perplexity)
- `image-generator` — generate AI images via text-to-image CLI
- `image-reviewer` — analyze images, screenshots, and visual artifacts
- `toolsmith` — create and tune agents, commands, and skills

Full definitions: `.opencode/agent/*.md` | Inventory: [.opencode/README.md](.opencode/README.md)

## Commands

| Command | Purpose |
|---------|---------|
| `/apply-review-feedback` | Classify and apply accepted PR/MR review feedback locally |
| `/bootstrap` | Scaffold ADOS artifacts for an existing project |
| `/plan-change` | Interactive planning session (prep context for /write-spec) |
| `/write-spec <ref>` | Generate change specification |
| `/write-test-plan <ref>` | Generate test plan |
| `/write-plan <ref>` | Generate implementation plan |
| `/run-plan <ref>` | Execute plan phases |
| `/review <ref>` | Review change vs spec/plan |
| `/review-deep <ref>` | Deep review with stronger reasoning model |
| `/review-remote` | Review open PR/MR diff and optionally publish findings |
| `/sync-docs <ref>` | Reconcile system docs from a change |
| `/check` | Run quality gates (no fixes) |
| `/check-fix` | Run quality gates and fix failures |
| `/commit` | Create one Conventional Commit |
| `/pr` | Create/update PR/MR |
| `/plan-decision` | Interactive architecture decision session |
| `/write-decision` | Generate Decision Record (ADR/PDR/TDR/BDR/ODR) |
| `/design` | Generate/update visual design assets |

Full definitions: `.opencode/command/*.md`

## Using the system

**Autopilot** (recommended) — `@pm` orchestrates all 10 phases:

```
@pm deliver change GH-456
```

**Manual** — you trigger each step:

```
/plan-change → /write-spec <ref> → /write-test-plan <ref> → /write-plan <ref>
→ /run-plan <ref> → /sync-docs <ref> → /review <ref> → /check → /pr
```

Guide: [doc/guides/opencode-agents-and-commands-guide.md](doc/guides/opencode-agents-and-commands-guide.md)

## Extending the system

When adding or modifying agents, commands, or skills:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juliusz-cwiakalski/agentic-delivery-os](https://github.com/juliusz-cwiakalski/agentic-delivery-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
