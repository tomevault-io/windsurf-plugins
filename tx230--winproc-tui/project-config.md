---
trigger: always_on
description: This document describes operating rules for AI coding agents (Codex, Cursor, and similar tools) working in this repository. The maintainer does not need to follow it as a checklist; durable product rules live in `README.md`, `README.ja.md`, and the documents under `docs/`.
---

# AGENTS.md

This document describes operating rules for AI coding agents (Codex, Cursor, and similar tools) working in this repository. The maintainer does not need to follow it as a checklist; durable product rules live in `README.md`, `README.ja.md`, and the documents under `docs/`.

This repository is the development repository for `winproc-tui`.
`winproc-tui` is a TUI process investigation tool for Windows 11 x64. It uses Rust 2024 edition, ratatui, crossterm, Windows APIs, PDH, DXGI, and sysinfo.

## Read Before Working

This repository has specifications under `docs/`. Before changing implementation or explanations, read the documents relevant to the requested work.

- `docs/metrics.md`: Metrics, data sources, display formats, CPU% semantics, sampling frequency, and recording logs.
- `docs/architecture.md`: Architecture, responsibility boundaries, data flow, major types, and testing policy.
- `docs/release-workflow.md`: Release tagging, packaging, and GitHub Release procedure.
- `README.ja.md`: Japanese user-facing overview.
- `README.md`: English user-facing overview for GitHub. Keep it synchronized with `README.ja.md`.

Prefer the current implementation under `src/` over old notes or guesses.
If the specifications and implementation conflict, inspect the implementation first and update the specifications if needed.

## Repository Policy

- Store text files as UTF-8 without BOM, using LF line endings. Keep this aligned with `.gitattributes`.
- This project is Windows-only. Do not add abstractions or explanations that assume Linux / macOS support unless explicitly requested.
- This is a personal project. Unsolicited pull requests from external contributors are not accepted. Use GitHub Issues for feedback and feature requests.
- `docs/` is Git-managed primary information for specifications, architecture, metrics, and release workflow. When implementation or specifications change, update the related documents in the same work item.
- `logs/` and `notes/` are local-only paths ignored by `.gitignore`. Do not treat them as publishable artifacts unless the user explicitly says so.
- Local-only work that changes only ignored paths such as `notes/` or `logs/` does not require an agent branch or commit.
- Existing uncommitted changes may be user work. Do not revert changes you did not make.
- Keep changes as small as practical. Avoid opportunistic large refactors and unrelated formatting churn.
- Keep maintained specifications under `docs/` in English.
- Keep Japanese documentation limited to `README.ja.md` unless the user explicitly asks otherwise.
- In `README.ja.md`, prefer natural, readable Japanese over literal translation or unnecessary English mixing.

## Documentation Workflow

- In general, work on the change requested by the user. If the user selects a GitHub Issue, work on exactly that one issue.
- Before implementing, read the target issue or request and related specifications. Do not mix requirements, design, and implementation instructions.
- If metrics, data sources, display formats, or recording log values change, update `docs/metrics.md`.
- If internal structure, responsibility boundaries, data flow, major types, or testing policy change, update `docs/architecture.md`.
- If user-facing behavior changes, update Help, Footer, README, tests, and source as appropriate.
- If a technical choice needs durable context, keep it in the related specification, architecture document, or GitHub Issue.
- Do not create or update repository-local backlog files under `docs/backlog/`; use GitHub Issues for backlog tracking.

## Commit Rules

- Use English Conventional Commits for commit messages.
- Keep commits scoped. Do not include unrelated dirty files or local-only artifacts.
- When a coherent unit of AI work is complete, commit it promptly.
- Do not commit ignored local-only files such as `notes/` or `logs/` unless the user explicitly asks to track them.
- When committing implementation work, include the related specification, metric, and architecture updates in the same commit if they describe the same behavioral change.
- Reference the relevant GitHub Issue in the commit message or maintainer-requested pull request when useful.

## Branch Workflow Rules

These branch / commit / push rules apply to AI agents. The maintainer usually integrates work locally; open a pull request only when the maintainer explicitly asks for one.

- Treat `main` as the stable default branch. Do not use it for experiments or multi-step work.
- AI agents must work on an `agent/<short-topic>` branch for tracked repository changes.
- Prefer a branch name that describes the work, for example `agent/help-dialog-copy` or `agent/branch-workflow-docs`.
- If the human gives a branch name, use the human-specified name instead of inventing one.
- Use `agent/YYYYMMDD-HHMM` only as a fallback when there is no clear topic name or when the human explicitly asks for a timestamp-only branch.
- AI agents must not commit to `main` unless the user explicitly instructs them to do so.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TX230/winproc-tui](https://github.com/TX230/winproc-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
