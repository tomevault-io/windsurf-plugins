---
trigger: always_on
description: Manage Trunk-Based Development workflows using the tbdflow CLI. Use this skill to create short-lived branches, make standardised commits, sync with trunk, merge completed work, and generate changelogs.
---


# tbdflow Skill

## Overview

This skill enables an AI agent to manage a **Trunk-Based Development (TBD)** workflow using the `tbdflow` CLI (v0.18.2).

The skill exists to:

- Enforce short-lived branches
- Standardise commits
- Reduce Git decision-making
- Maintain a fast, safe path back to trunk (`main`)

`tbdflow` is the **only interface** the agent should use for Git workflow actions covered by this skill.

---

## When to Use

Use this skill when the user wants to:

- Start work on a task, ticket, or feature
- Commit staged changes
- Sync with trunk or check repository status
- Merge completed work back to `main`
- See what has changed since the last release
- Log a design decision, failed attempt, or reasoning pivot

Typical trigger phrases include:

- "Start working on…"
- "Commit this"
- "Merge my work"
- "Sync me up"
- "What's new?"
- "Note that…" / "Log this decision"

---

## When *Not* to Use

Do **not** use this skill to:

- Create or manage long-lived branches
- Perform manual Git commands outside `tbdflow`
- Rewrite commit history
- Perform interactive rebases
- Merge without explicit user intent

If an action cannot be performed via `tbdflow`, explain the limitation instead of falling back to raw Git commands.

---

## Prerequisites

Before invoking any `tbdflow` command, the agent **must** verify that the CLI is installed and accessible.

### Installation Check

Run the following to confirm availability:

```bash
command -v tbdflow && tbdflow --version
```

If `tbdflow` is **not found**, the agent must attempt to install it automatically using one of the strategies below.

### Installation Strategies (in order of preference)

**1. Cargo Install (requires Rust toolchain)**

If `cargo` is available on the system:

```bash
cargo install tbdflow
```

This downloads the latest release from [crates.io](https://crates.io/crates/tbdflow) and compiles it locally.

**2. GitHub Releases (pre-built binary)**

If `cargo` is not available but `curl` is:

```bash
curl -fsSL https://github.com/cladam/tbdflow/releases/latest/download/tbdflow-$(uname -m)-$(uname -s | tr '[:upper:]' '[:lower:]') -o /usr/local/bin/tbdflow
chmod +x /usr/local/bin/tbdflow
```

Adjust the binary path if `/usr/local/bin` is not writable (e.g. use `~/.local/bin`).

**3. Manual Prompt**

If neither strategy is viable, inform the user:

> `tbdflow` is not installed. Please install it using one of:
> - `cargo install tbdflow`
> - Download a binary from https://github.com/cladam/tbdflow/releases
>
> See the README for details.

### Post-Install Verification

After installation, always confirm:

```bash
tbdflow --version
```

If the version is outdated, suggest:

```bash
tbdflow update
```

---

## Instructions

Follow the instructions below exactly. Each capability defines intent, constraints, and decision rules.

---

### 1. Standardised Committing

**Intent**
Create a structured, conventional commit on trunk or a short-lived branch.

**Staging Behaviour**

* `tbdflow` automatically stages the relevant changes when committing
* The agent must **not** run `git add` or any raw Git staging commands
* No explicit staging step is required from the user or the agent

**Preconditions**

* The working tree contains changes intended for commit
* No unresolved merge conflicts are present

**Command**

```bash
tbdflow commit -t <type> [-s <scope>] -m "<message>" [--issue <issue>] [-b]
```

**Decision Rules**

* Allowed commit types:

    * `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `build`, `ci`, `perf`, `revert`, `style`
* Never invent new commit types
* If no type is specified:

    * Default to `chore` unless behaviour changes
* DoD Checklist: If a `.dod.yml` file exists in the project root and `--no-verify` is not passed, an interactive
  checklist will appear. Unchecked items will result in a `TODO:` footer being appended to the commit message.
* Use `-b` / `--breaking` if the change introduces breaking behaviour
* Use `--issue` when the user references a ticket ID (JIRA, GitHub, etc.)

**Use This When**

* The user says “commit”, “save this”, or “check this in”
* The user describes completed work ready to be recorded

---

### 2. Creating Short-Lived Branches

**Intent**
Start a new unit of work in a short-lived branch that will be merged back to trunk quickly.

**Preconditions**

* Working tree is clean or safely stashed
* The task is not exploratory or long-running

**Command**

```bash
tbdflow branch -t <type> -n <name> [--issue <issue>] [-f <from_commit>]
```

**Decision Rules**

* Branch naming follows:

    * `<type>/<name>` or
    * `<type>/<issue>-<name>`
* If the user provides a task description:

    * Slugify it for the `-n` parameter
* Use `--issue` when a ticket ID is available
* Use `-f` only if the user explicitly asks to branch from a non-HEAD commit

**Use This When**

* The user says “start working on…”
* A task requires isolation before merging to `main`

---

### 3. Workflow Completion & Integration

**Intent**
Safely merge completed work back into trunk and clean up the branch.

**Preconditions**

* All intended commits are complete

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cladam/tbdflow](https://github.com/cladam/tbdflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
