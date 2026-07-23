---
trigger: always_on
description: Master context document for AI coding agents contributing to this repository.
---

# Agent Guidelines for tfutils/tfenv

Master context document for AI coding agents contributing to this repository.
Every agent MUST read this file before starting work.

---

## Project Overview

tfenv is a Terraform version manager written in Bash, modelled after rbenv.
~2.5k LOC across `bin/`, `lib/`, `libexec/`, `test/`, and `share/`.

- **Language:** Bash (no shellcheck — deliberate choice; see
  [ADR-0003](docs/adr/0003-no-shellcheck.md))
- **Default branch:** `master`
- **Current release:** v3.2.0 (April 2026)
- **Maintainer:** Mike Peachey / Jaz (@Zordrak)
- **Minimal dependencies:** bash, curl, grep/ggrep, sort, unzip
- **License:** MIT

## Repository Structure

```
bin/              Entry points (terraform shim, tfenv command)
lib/              Shared libraries sourced by multiple scripts
libexec/          Subcommands (tfenv-install, tfenv-use, etc.)
test/             Integration tests (download real Terraform binaries)
share/            Static assets (HashiCorp PGP keys)
.github/          CI workflows, agent definitions, issue templates
  agents/         Agent definition files (.agent.md)
  instructions/   Auto-loaded coding standards
  ISSUE_TEMPLATE/ Structured issue forms
docs/
  adr/            Architecture Decision Records
```

---

## Core Principles

### 1. Standalone Script Execution

Every `libexec/` script contains its own boilerplate for resolving
`TFENV_ROOT` and sourcing helpers. This is **intentional** — each script
must be executable in isolation. Do NOT refactor this into a shared loader.
See [ADR-0002](docs/adr/0002-standalone-script-execution.md).

### 2. Worktree-First

ALL code changes MUST happen in a git worktree under `.worktrees/`. The main
working tree is shared by all agents and the human — modifying it directly
risks conflicts. `.worktrees/` is gitignored.

```bash
# Create a worktree for your work
git worktree add .worktrees/fix-123 -b fix/123-description master

# Do all work inside the worktree
cd .worktrees/fix-123

# Clean up when done (after PR is created)
cd /path/to/main/tree
git worktree remove .worktrees/fix-123
```

**Exception:** Read-only operations (searching, reading files, running tests)
may use the main working tree.

### 3. Quality Over Speed

- Run the test suite before committing
- Read the diff before pushing
- Check your own work before calling it done
- Fix problems when you find them — do not defer

### 4. Minimal Dependencies

The project deliberately has minimal dependencies: bash, curl, grep/ggrep,
sort, and unzip. Do NOT add new external tools or packages without explicit
approval from the maintainer.

### 5. No ShellCheck

The project does not use ShellCheck. This is a deliberate architectural
decision. Do NOT add shellcheck directives, configs, or CI steps.
See [ADR-0003](docs/adr/0003-no-shellcheck.md).

---

## Branch and PR Workflow

1. **Never commit directly to `master`.** Always use a feature branch.
2. **Branch naming:** `fix/<short-description>` for bugs,
   `feat/<short-description>` for features, `chore/<short-description>`
   for maintenance. Reference the issue number where applicable
   (e.g. `fix/406-unbound-requested-variable`).
3. **One logical change per branch/PR.** Do not bundle unrelated fixes.
4. **PRs target `master`.** There is no develop or staging branch.
5. **Merge strategy:** The repo uses merge commits (not squash).
   Do not force-push or rebase shared branches.

## Commit Messages

Freeform — no conventional commits standard is enforced. Be descriptive.
Reference issue numbers where applicable (e.g. `Fix #406: ...`).

Historical examples:
- `Fix realpath not available on macOS`
- `Cope with different line endings in .terraform-version`
- `Reduce duplication, and add safety`

---

## Claim Protocol

Before starting work on any issue, an agent MUST claim it:

1. Add the `agent:in-progress` label to the issue
2. Post a comment: `Claimed by <agent-name>. Working on this.`
3. If the label is already present, the issue is claimed — do NOT work on it

This acts as a distributed lock preventing duplicate effort when multiple
agents run concurrently.

When work is complete (PR created), remove `agent:in-progress` and add
`agent:review-requested`.

```bash
# Claim an issue
gh issue edit NNN --add-label 'agent:in-progress'
gh issue comment NNN --body 'Claimed by <agent-name>. Working on this.'

# Release after PR created
gh issue edit NNN --remove-label 'agent:in-progress' --add-label 'agent:review-requested'
```

---

## GitHub CLI Strategy

The `gh` CLI is the **primary and load-bearing** interface to GitHub for all
agents. Use `gh api repos/tfutils/tfenv/...` for operations not covered by
high-level `gh` commands.

The GitHub MCP server may supplement `gh` where proven reliable, but must
never be on the critical path.

---

## Work Type Ownership

Each work type is owned by a specific agent. If you receive a request that
belongs to a different agent, say so and stop.

| Work Type | Owning Agent | Description |
| --------- | ------------ | ----------- |
| Autonomous delivery orchestration | `developer` | Assess board, dispatch specialists |
| Bug hunting and auditing | `bug-finder` | Find defects, file structured issues |
| Bug fixing | `bug-fixer` | Implement fixes with tests |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tfutils/tfenv](https://github.com/tfutils/tfenv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
