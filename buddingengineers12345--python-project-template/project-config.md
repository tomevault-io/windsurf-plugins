---
trigger: always_on
description: This repository is a **Copier template repository** (a meta-project). Running Copier against
---

# python_project_template — Claude Project Knowledge

## What this repo is

This repository is a **Copier template repository** (a meta-project). Running Copier against
this repo **generates** a new Python project by rendering the `template/` directory into a
destination folder.

> [!WARNING]
> Copier can run **template tasks** during `copier copy`/`copier update`. Only use the
> `--trust` flag with templates you trust.

## Directory structure

```
.
├── template/                 # Jinja2 source files that Copier renders
│   ├── src/{{ package_name }}/   # Generated package source (common/, core.py, cli.py…)
│   ├── tests/                # Generated project test suite
│   ├── .claude/              # Claude hooks/commands/rules/skills for generated projects
│   ├── .github/workflows/    # Generated CI/CD workflows
│   └── …                    # pyproject.toml.jinja, justfile.jinja, CLAUDE.md.jinja, …
├── tests/                    # pytest suite for this meta-repo (see tests/CLAUDE.md)
│   ├── constants.py          # REPO_ROOT / TEMPLATE_ROOT / COPIER_YAML for nested test modules
│   ├── conftest.py           # top-level shared fixtures
│   ├── unit/                 # fast isolated script tests
│   ├── integration/          # Copier copy/update integration suite
│   └── e2e/                  # end-to-end tests (placeholder)
├── scripts/                  # Automation scripts for CI or local tasks (see scripts/CLAUDE.md)
│   ├── repo_file_freshness.py    # Git-based freshness dashboard (→ docs/ + assets/)
│   ├── bump_version.py           # PEP 440 version bumper (patch/minor/major)
│   ├── check_root_template_sync.py  # Root ↔ template parity (workflows, settings, recipes)
│   ├── pr_commit_policy.py       # PR title/body + commit message policy (CI)
│   └── sync_skip_if_exists.py    # Sync copier.yml _skip_if_exists with template paths
├── .claude/                  # Claude Code hooks, commands, and rules for THIS meta-repo
│   ├── settings.json         # Hook registrations and permission allow/deny lists
│   ├── hooks/                # Shell hook scripts (see hooks/README.md)
│   ├── commands/             # Slash command prompts (/review, /generate, /release, …)
│   └── rules/                # AI rules (common/, python/, jinja/, bash/, yaml/, copier/)
├── docs/                     # Markdown output folder (repo_file_status_report.md, etc.)
├── assets/                   # Freshness JSON artifacts (file_freshness.json, etc.)
├── .github/                  # Meta-repo GitHub Actions workflows
├── copier.yml                # Template prompts, computed vars, and post-gen tasks
├── justfile                  # Task runner (use `just` not raw commands)
├── pyproject.toml            # Dev deps for THIS repo (not for generated projects)
├── .pre-commit-config.yaml   # Pre-commit hooks for meta-repo
└── uv.lock                   # Committed lockfile — never delete
```

## How to set up the development environment

```bash
uv sync --frozen --extra dev   # install all dev deps from the lockfile
just precommit-install         # register git hooks
```

Prerequisites: Python 3.11+, `uv`, `just`, `git`.

## Common development commands

| Task | Command |
|---|---|
| List recipes (default) | `just` or `just default` |
| Run all tests | `just test` |
| Run tests in parallel | `just test-parallel` |
| Run slow tests only | `just slow` |
| Fast unit tests (no slow/integration) | `just test-fast` |
| Integration tests only | `just test-integration` |
| Tests for changed files only | `just test-changed` |
| Verbose tests | `just test-verbose` |
| Full debug test output | `just test-debug` |
| Re-run last failed tests | `just test-lf` |
| Re-run last failed tests (max verbosity) | `just test-failed-verbose` |
| Stop on first test failure | `just test-first-fail` |
| CI-style tests + coverage XML | `just test-ci` |
| Coverage report | `just coverage` |
| Lint | `just lint` |
| Lint changed files only | `just lint-changed` |
| Format | `just fmt` |
| Format check (read-only) | `just fmt-check` |
| Auto-fix lint issues | `just fix` |
| Type check | `just type` |
| Docstring check | `just docs-check` |
| MkDocs recipes (generated projects only) | `just docs-help` |
| Pre-merge review (fix + lint + type + docs) | `just review` |
| Full CI locally (fix → check) | `just ci` |
| Read-only CI check (no auto-fix) | `just check` |
| Run pre-commit on all files | `just precommit` |
| Register git hooks | `just precommit-install` |
| Interactive conventional commit (Commitizen) | `just cz-commit` |
| Sync deps after lockfile change | `just sync` |
| Upgrade all deps | `just update` |
| Check for outdated dependencies | `just deps-outdated` |
| Verify lockfile integrity | `just lock-check` |
| Dependency security audit | `just audit` |
| Install all deps + pre-commit | `just install` |
| One-command developer onboarding | `just bootstrap` |
| Diagnose environment | `just doctor` |
| Generate freshness dashboard | `just freshness` |
| Root ↔ template sync validation | `just sync-check` |
| Suggested PR title + body (PR policy) | `just pr-draft` |
| Clean build artifacts | `just clean` |
| Build distribution | `just build` |
| Validate built distribution | `just check-dist` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buddingengineers12345/python_project_template](https://github.com/buddingengineers12345/python_project_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
