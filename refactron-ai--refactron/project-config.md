---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Operations scaffolding (`.claude/` + repo root)

Route through these resources instead of redoing the work each session.

**Subagents (`.claude/agents/`)**: 12 senior personas (10+ yrs framing) for delegation. Each declares its peers under "Hand-offs."

| Agent                       | Use for                                                 |
| --------------------------- | ------------------------------------------------------- |
| `delivery-lead`             | Issue shaping, sizing, definition of done               |
| `principal-engineer`        | Architecture / locked-contract calls / breaking changes |
| `staff-code-reviewer`       | Adversarial pre-merge review                            |
| `security-engineer`         | Threat modeling, sidecar safety, supply chain           |
| `offensive-security-engineer` | Red-team: build working exploits against the engine    |
| `python-sidecar-specialist` | Verify-check sidecars, stdin/stdout protocol, refusals  |
| `typescript-architect`      | Verify engine, ts-morph, ESM, type-level safety         |
| `release-manager`           | Semver, changelog, npm + PyPI publish                   |
| `test-engineer`             | Red-first proof, fixtures, snapshot discipline          |
| `dx-engineer`               | Verdict output, CLI ergonomics, error messages          |
| `documentation-engineer`    | README, mdx, changelog tone, migrations                 |
| `performance-engineer`      | Throughput, sidecar latency, profiling                  |

Review and analysis roles (`staff-code-reviewer`, `principal-engineer`, `security-engineer`, `offensive-security-engineer`, `performance-engineer`) hold read-only tool grants: a reviewer that can edit stops arguing with itself and starts fixing, which is how review findings get quietly absorbed instead of reported.

**Issue-first workflow**

Work starts as an issue. The branch is named `<type>/<number>-<slug>`, and the PR closes the issue.

1. `/issue` shapes an intent into a well-formed issue (problem, evidence, acceptance criteria, non-goals) via `delivery-lead`, and files it.
2. `/start <number>` reads the issue, creates the branch, plans the approach, and begins test-first.
3. `/review` runs the three-lens parallel review (correctness, architecture, tests) against the current branch.
4. `/ship` verifies the work against its issue, runs the review, and opens the PR with `Closes #<n>`.

Use `/ship` instead of `gh pr create`. A branch with no issue behind it is work nobody agreed to.

**Slash commands (`.claude/commands/`)**

- `/issue`: shape an intent into a well-formed GitHub issue and file it (routes through `delivery-lead`).
- `/start <number>`: pick up an issue, branch for it, plan, begin test-first.
- `/review`: three-lens adversarial review of the current branch (correctness, architecture, tests), run in parallel.
- `/ship`: the pre-PR gate. Verify against the issue, run the review, open the PR.
- `/check-locked`: verify the locked-files invariant on the current branch.

**Rules**

- `CODE_STYLE.md`: concrete TS + Python rules
- `COMMIT_CONVENTIONS.md`: Conventional Commits + this repo's scope vocabulary + authorship rules

**Operations & navigation**

- `ARCHITECTURE.md`: engines, locked surfaces, pipeline, invariants
- `GLOSSARY.md`: verdict, gate, shadow tree, attribution, sidecar, and the rest
- `RUNBOOK.md`: release, rollback, CVE response, snapshot regeneration

**Templates**: `docs/prd/_template.md`, `docs/plans/_template.md`, `dev-docs/decisions/_template.md` (each links real examples in this repo).

**Settings & hooks (`.claude/settings.json`, `.claude/hooks/`)**: team-wide permissions plus three live hooks:

- `block-dangerous-bash.sh` (PreToolUse:Bash): blocks `--no-verify`, `--force` push, `git reset --hard`, real `npm publish`, broad `rm -rf`
- `block-locked-file-writes.sh` (PreToolUse:Write|Edit): blocks edits to `src/contracts.ts`
- `auto-format.sh` (PostToolUse:Write|Edit): runs prettier on supported files so format:check stays green
- `.githooks/commit-msg`: enforces COMMIT_CONVENTIONS (Conventional Commits shape, no AI names, no co-author trailers, 72-char subject cap). Activated by `npm install` via the `prepare` script.

**Ownership**: `.github/CODEOWNERS` auto-requests review on locked files, ADRs, ops scaffolding, and release-critical files.

When a non-trivial task arrives, the first move is usually: find or file the issue, pick the matching subagent, read the relevant rule doc, and start from the template, not from scratch.

## Common Commands

```bash
npm run build              # tsc --project tsconfig.build.json (chmods dist/cli/index.js)
npm run typecheck          # tsc --noEmit
npm run lint               # eslint, --max-warnings 0 (warnings fail)
npm run format             # prettier --write
npm run format:check       # CI-style prettier check

npm test                   # vitest run (full suite)
npm run test:watch         # vitest in watch mode
npm run test:coverage      # vitest with v8 coverage

# Single test file / single case:
npx vitest run tests/unit/<file>.test.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Refactron-ai/refactron](https://github.com/Refactron-ai/refactron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
