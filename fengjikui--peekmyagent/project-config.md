---
trigger: always_on
description: This file is the operational contract for coding agents working on peekMyAgent from macOS, Windows, or Linux. It applies to Codex, Claude Code, and any other automated contributor.
---

# Coding Agent Collaboration Covenant

This file is the operational contract for coding agents working on peekMyAgent from macOS, Windows, or Linux. It applies to Codex, Claude Code, and any other automated contributor.

The words **MUST**, **SHOULD**, and **MAY** are normative.

## 1. Source Of Truth

- `origin/main` is the only shared source of truth.
- A local chat, handoff note, database, generated report, or unpushed commit is not shared project state.
- Every validation report MUST name an exact commit SHA. Do not report that "the latest code" passed.
- Read [the current architecture](docs/architecture.md), [the refactoring roadmap](docs/refactoring-roadmap.md), and the relevant source before changing behavior.
- Inspect current files and tests. Do not implement from an old conversation summary when the repository can answer the question.

## 2. Contributor Roles

### Primary contributor

Unless the project owner assigns a different lead, the Codex agent working from the owner's primary macOS workspace is the coordinating primary contributor. It coordinates architecture, sequencing, integration, and release readiness. This is a coordination role, not permanent ownership of every subsystem.

The primary contributor:

- keeps `main` coherent and the roadmap current;
- assigns an exact SHA and validation scope to platform agents;
- decides cross-module architecture after reviewing platform evidence;
- makes sure a platform fix becomes a portable regression test;
- does not overrule a reproducible platform failure merely because another platform passes.

### Platform contributor

A Windows, Linux, or macOS agent owns the quality of the platform it is currently validating. It MAY become the main contributor for a scoped feature or subsystem.

A platform contributor:

- validates the assigned SHA on a real machine;
- records OS version, architecture, shell, Node/npm versions, commands, and results;
- fixes failures on a dedicated branch rather than directly on `main`;
- prefers shared abstractions over platform-specific conditionals;
- reports a clean pass without creating an empty or report-only commit.

Responsibility follows the task and evidence, not a permanent hierarchy.

## 3. Synchronization Cadence

### Before every work session

Every coding agent MUST:

```bash
git fetch origin
git status --short --branch
git rev-parse HEAD
git rev-parse origin/main
```

- Start new work from the assigned SHA or current `origin/main`.
- If the working tree contains changes from another contributor, do not discard or overwrite them.
- Fetch again immediately before pushing or opening a pull request.

### Continuous automated validation

- Every pull request and every push to `main` or `codex/**` MUST run the GitHub Actions macOS, Windows, and Linux matrix.
- Hosted CI runs for every change. Do not batch CI until several large changes have accumulated.

### Real-machine validation

- Platform-sensitive changes SHOULD be validated on real Windows and Linux machines within one working day, and MUST be validated before a release.
- During active development, each real platform SHOULD run a checkpoint at least weekly even when no single change was classified as high risk.
- Every release candidate MUST be validated on the exact candidate SHA on macOS, Windows, and Linux.
- Documentation-only changes do not require an extra real-machine session beyond hosted CI.

## 4. Platform Risk Classification

Treat a change as **high platform risk** when it touches any of these areas:

- CLI parsing or wrapper lifecycle under `bin/`;
- process spawning, signals, shells, paths, environment variables, or permissions;
- installation, global npm commands, maintenance, or uninstall;
- ports, loopback binding, Capture Proxy, OTel, or daemon lifecycle;
- SQLite storage, file permissions, import/export, or cleanup;
- Agent configuration discovery or reversible patching;
- `package.json`, GitHub workflows, or platform release gates.

High-risk changes require:

1. focused deterministic smoke tests;
2. the full platform profile on the contributor's host;
3. hosted three-platform CI;
4. real-machine validation on affected platforms before release.

Viewer-only JavaScript, CSS, copy, or documentation changes still use hosted CI, but normally do not require immediate real-machine validation unless they alter browser, filesystem, or local API behavior.

## 5. Branch And Commit Rules

- Platform agents MUST work on a branch such as `codex/windows-<task>`, `codex/linux-<task>`, or `codex/macos-<task>`.
- Do not push directly to `main` from a validation machine.
- Keep each commit focused on one behavior, bug, or refactoring boundary.
- Do not mix formatting churn, generated artifacts, personal notes, or unrelated cleanup into a fix.
- Use semantic commit subjects that describe observable intent.
- Before committing, inspect `git diff`, `git status`, and the exact files being staged.
- Rebase or merge the current `origin/main` before final validation when the branch is stale. Never force-push over another contributor's work without explicit coordination.

## 6. Implementation Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fengjikui/peekMyAgent](https://github.com/fengjikui/peekMyAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
