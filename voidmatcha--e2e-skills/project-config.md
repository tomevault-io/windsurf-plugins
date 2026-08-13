---
trigger: always_on
description: › **Reading this in Claude Code?** See also `CLAUDE.md` if present. This file is read by Codex and other agents that follow the `AGENTS.md` convention.
---

› **Reading this in Claude Code?** See also `CLAUDE.md` if present. This file is read by Codex and other agents that follow the `AGENTS.md` convention.

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, and other AGENTS.md-compatible hosts) working in this repository.

## Repository Overview

`e2e-skills` is a bundle of four Agent Skills for end-to-end test work on Playwright and Cypress projects:

- `playwright-test-generator` — generates Playwright E2E tests from scratch (coverage-gap analysis → live-browser exploration → approval gate → review).
- `e2e-reviewer` — static review of existing Playwright/Cypress specs against 24 anti-patterns grouped P0/P1/P2.
- `playwright-debugger` — root-cause diagnosis from `playwright-report/`.
- `cypress-debugger` — root-cause diagnosis from `cypress/reports/` (mochawesome / JUnit).

The repo doubles as a Claude Code plugin (`.claude-plugin/`), a Codex plugin (`.codex-plugin/`), a cross-agent skill source via the `skills` CLI, and a standalone scanner (`skills/e2e-reviewer/scripts/scan.sh`).

## Verification gate (must pass before commit)

```
[ ] /bin/bash -p scripts/ci/ci-local.sh          # review checks + drift smoke checks + 0 P0 smell hits
[ ] /bin/bash -p scripts/ci/pre-push-security.sh # secrets and credential leak guard
```

`ci-local.sh` is the single source of truth for the repository CI mirror
(shell syntax, security, parity, evals, public skill surface, framework scope,
link integrity, docs orphan check, language, and the E2E smell scan).
`ci-local.sh` reaches the security gate through `review.sh`; the explicit
`pre-push-security.sh` command above reruns that mandatory gate at the
pre-push boundary. If you change a CI check, update `ci-local.sh` first.

## Directory Layout

```
.
├── .claude-plugin/         # Claude Code plugin + marketplace manifests
│   ├── plugin.json
│   └── marketplace.json
├── .codex-plugin/          # Codex plugin manifest (interface display surface)
│   └── plugin.json
├── AGENTS.md               # This file (cross-agent canonical guide)
├── agents/                 # Claude Code subagents (plugin-install only): read-only
│   ├── e2e-finding-verifier.md    # adversarially verify ONE reviewer finding
│   └── e2e-failure-classifier.md  # classify ONE failure into F1–F15
├── .codex/agents/          # Codex-native TOML ports of the two subagents (optional)
│   ├── e2e-finding-verifier.toml   # behavior-synced with agents/*.md; guarded by SP5
│   └── e2e-failure-classifier.toml
├── benchmarks/             # immutable public benchmark evidence + adjudication ledgers
├── skills/                 # Four Agent Skills (the public surface)
│   ├── playwright-test-generator/
│   │   ├── SKILL.md        # Required: skill frontmatter + body
│   │   ├── best-practices.md
│   │   ├── code-rules.md
│   │   ├── evals/evals.json
│   │   └── agents/openai.yaml
│   ├── e2e-reviewer/
│   ├── playwright-debugger/
│   └── cypress-debugger/
├── scripts/
│   ├── ci/                 # CI parity, security, eval-metadata checks
│   ├── dev/                # contributor reinstall + git hook setup
│   ├── evals/              # labeled reviewer holdout, live runners, executable fixtures
│   ├── hooks/              # local git hooks
│   ├── pr-preflight.sh     # seven-stage preflight for upstream E2E-fix PRs
│   ├── verify-fixes.sh     # post-bulk-fix verification (sed-artifact AST detection)
│   └── validate-evals.sh
├── docs/                   # Open-source assets (taxonomy, case studies, scope)
├── README.md
└── CHANGELOG.md
```

Each `skills/<name>/SKILL.md` is the contract. Everything in the skill body should be **task-actionable instructions for the agent**, not narrative documentation; supporting reference material (long tables, framework references) goes in sibling `.md` files and is read on demand.

## Conventions

- **Skill names**: kebab-case, must match the directory name and the `name:` in SKILL.md frontmatter.
- **SKILL.md frontmatter**: `name`, `description`, `license`, `metadata: { author, version }`. The description is the trigger surface — pack synonyms and the user's likely phrasing.
- **Pattern IDs**: 24 stable anti-pattern entries (`#1`–`#23` plus `#3b`) with P0/P1/P2 severity. IDs are stable; do not renumber. Severity rationale: P0 = silent always-pass, P1 = poor diagnostics, P2 = maintenance.
- **Failure category IDs**: 15 codes (`F1`–`F15`) used by both debuggers. Codes are stable.
- **JUSTIFIED comments**: `// JUSTIFIED: <reason>` on the line above (or above the enclosing block / multi-line chain) suppresses scanner findings. Suppress for documented intent, never to hide a real finding.
- **Severity-first organization**: tables in SKILL.md, README, and `docs/e2e-test-smells.md` group by P0/P1/P2 in the same order.
- **English-only public surface**: SKILL.md, README, and `docs/` are English. CI enforces this (`Language` check). Sanctioned exception: root-level `README.<lang>.md` translations (`README.ko.md`, `README.ja.md`, `README.zh-cn.md`) and the language-switcher line in `README.md` that links to them. `README.md` (English) is canonical; translations follow it.

## Frameworks in Scope


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voidmatcha/e2e-skills](https://github.com/voidmatcha/e2e-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
