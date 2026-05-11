---
trigger: always_on
description: Before ANY action in this session:
---

# CLAUDE.md

## STOP — READ THIS FIRST

Before ANY action in this session:
1. Read this entire file
2. Read docs/CONTRIBUTING.md
3. Acknowledge the rules that apply

Do not proceed until you've done this.

> **Your failure mode:** You optimize for completion. This drives you to batch—do many things, report success. Resist this. Do less. Verify more. When something breaks, understand first. A fix you don't understand is a timebomb.
>
> **Before actions that could fail:** State what you expect. After: compare to reality. (See `docs/TROUBLESHOOTING.md#prediction-protocol`)
>
> **Before changing working code:** Run `git log -1 --oneline <file>` and `git diff HEAD -- <file>` first. If the code was working before, understand WHY before touching it. Revert fast if your fix doesn't work on first try.

---

## First Session?

**See `docs/BOOTSTRAP.md`** for initial project setup (Git, quality gates, tooling).

Return here after bootstrap is complete.

---

## The 13 Rules

### Context Management
| # | Rule | Reference |
|---|------|-----------|
| 1 | **READ** docs/ files at session start AND after compaction | `docs/ARCHITECTURE.md`, `docs/CONTRIBUTING.md` |
| 2 | **RE-READ** this file when: after compaction, conversation is long, you're uncertain, or before any refactor | - |
| 3 | **RECALL** relevant docs before specialized tasks (testing, integrations, migrations) | See rule references below |

### Never Break Production
| # | Rule | Reference |
|---|------|-----------|
| 4 | **NEVER** ship refactors without diffing every endpoint/function against original | `docs/LESSONS-LEARNED.md` |
| 5 | **NEVER** exceed 300 lines per file - splits must improve architecture, not just move code | `docs/CONTRIBUTING.md#splits-must-improve-architecture` |
| 6 | **NEVER** bypass data source rules defined in architecture | `docs/ARCHITECTURE.md#data-sources` |
| 7 | **NEVER** widen test assertions to make failing tests pass - fix inputs or code instead | `docs/LESSONS-LEARNED.md` |

### Always Do
| # | Rule | Reference |
|---|------|-----------|
| 8 | **ALWAYS** run full test suite before declaring work complete | `docs/CONTRIBUTING.md#testing` |
| 9 | **ALWAYS** update docs BEFORE committing: CHANGELOG, TASKS, LESSONS-LEARNED if applicable | `docs/CONTRIBUTING.md#before-committing` |
| 10 | **ALWAYS** use shared utilities for common behaviors | `docs/CONTRIBUTING.md#shared-patterns` |
| 11 | **ALWAYS** verify dates are current year before writing timestamps | - |
| 12 | **CHECKPOINT** = `git commit` (not just TASKS.md) - commit before starting new features | `docs/CONTRIBUTING.md#session-discipline` |

### Governance
| # | Rule | Reference |
|---|------|-----------|
| 13 | **PROTECTED FILE** - CLAUDE.md changes require human approval. Show diff + provide `git commit --no-verify` command for user to run manually. Never run --no-verify yourself. | `docs/LESSONS-LEARNED.md` |

---

## Authority Hierarchy

When instructions conflict, higher authority wins:

1. **CLAUDE.md** (this file) - supreme
2. **docs/CONTRIBUTING.md** - code standards
3. **docs/ARCHITECTURE.md** - structural decisions
4. **All other docs** - reference material

---

## Failure Handling

| Situation | Required Response |
|-----------|-------------------|
| Tests fail | Diagnose root cause before changing any code (see `docs/TROUBLESHOOTING.md`) |
| Inventory drift | Update inventory file, never delete the test |
| File exceeds 300 lines | Split with architectural improvement, document tech debt if workaround |
| Pre-commit hook fails | Fix the issue, never bypass with `--no-verify` |
| CI fails | Fix locally, never modify CI to skip checks |

**Never disable quality gates to "make things pass."**

---

## Anti-Patterns (Explicit Prohibitions)

- **Do NOT** remove or skip tests to fix failures
- **Do NOT** widen expected values (e.g., adding status codes) to make tests pass
- **Do NOT** inline logic to avoid file splits
- **Do NOT** extract code just to hit line limits without improving architecture (metric gaming)
- **Do NOT** modify CI config to bypass checks
- **Do NOT** delete inventory entries to pass drift detection
- **Do NOT** use `any` types to silence TypeScript errors
- **Do NOT** comment out failing code instead of fixing it

---

## Quality Gates (Enforced Automatically)

These checks run on every commit via pre-commit hooks:

| Check | What It Catches | Fail Behavior |
|-------|-----------------|---------------|
| `typecheck` | Type errors, implicit any | Commit blocked |
| `lint` | Code style violations, unused vars | Commit blocked |
| `check:file-length` | Files > 300 lines | Commit blocked |
| `test` | Broken functionality, missing inventory entries | Commit blocked |

### Drift Detection (Inventory Tests)

Tests automatically fail if code doesn't match inventories:

| Change | Required Inventory Update |
|--------|---------------------------|
| Add/remove API endpoint | `tests/endpoint-inventory.json` |
| Add/remove component | `tests/component-inventory.json` |
| Add/remove module | `tests/module-inventory.json` |

**No code ships without passing all gates.**

---

## Quick Start (After Bootstrap)

```bash
# Development
[your dev command]

# Quality checks (run before ANY commit)
npm run precommit   # or: make check, cargo check, etc.

# Tests only
npm test
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madison-hutson/claude-project-scaffolding](https://github.com/madison-hutson/claude-project-scaffolding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
