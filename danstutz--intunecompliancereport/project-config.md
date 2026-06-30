---
trigger: always_on
description: <!-- markdownlint-disable MD013 -->
---

<!-- markdownlint-disable MD013 -->
# Repository Copilot Instructions (Repo-Wide Constitution)

**Version:** 1.6.20260629.1

## Metadata

- **Status:** Active
- **Owner:** Repository Maintainers
- **Last Updated:** 2026-06-29
- **Scope:** Repo-wide canonical instructions ("constitution") that govern all changes in this repository. This file is the authoritative source of truth for repository rules; all language-specific instruction files and agent entry points defer to it.
<!-- template-sync: begin markdown-reference-only -->
- **Related:** [Documentation Writing Style](instructions/docs.instructions.md)
<!-- template-sync: end markdown-reference-only -->

These instructions are authoritative for all changes in this repository.

## Source of Truth

Read **`docs/spec/requirements.md`** before making product, test, packaging, or documentation changes for the IntuneComplianceReport module. The specification's settled decisions, including the recommended repository layout, public command surface, repository metadata, and supported PowerShell editions, are authoritative.

If any repository instruction conflicts with `docs/spec/requirements.md`, **the spec wins** for product behavior and project identity.

## Protected Instruction Files

Instruction files and style guides are protected governance files. This rule applies to:

- The repo-wide constitution: `.github/copilot-instructions.md`
- Root agent entry points: `.hermes.md`, `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md`
- Cursor project rules under `.cursor/rules/`
- Modular instruction files under `.github/instructions/`

Agents **MUST NOT** create, edit, delete, rename, or otherwise change protected instruction files unless the repository owner or maintainer has directly and explicitly authorized the specific instruction-file change in the current task. Implied consent is insufficient.

Authorization **MUST NOT** be inferred from:

- An agent-generated plan, rubric, option analysis, or implementation strategy.
- A request to fix code, resolve review feedback, update documentation as needed, or keep files in sync.
- Pre-commit, formatting, linting, validation, or other cleanup work.
- An automated review loop, reusable prompt, or generic permission to make repository changes.

When an agent identifies a warranted instruction or style-guide update without explicit authorization, it **MUST** propose the change separately (for example, as a prompt or Open Question) and wait for explicit approval before editing protected files.

When explicit authorization is granted, keep protected instruction-file edits narrowly scoped, preserve the canonical source-of-truth hierarchy, and update related metadata and version fields according to [Documentation Writing Style](instructions/docs.instructions.md).

### Template Adoption and Stack Selection

Downstream repositories that keep only part of this template's language or tooling stack often need to update protected instruction files after deleting non-protected files. Use this order:

1. Perform non-protected cleanup first, such as deleting unused workflows, example source, tests, templates, and lint configuration.
2. Record the protected-file edits needed to remove references to deleted tools, workflows, hooks, validation commands, and language stacks.
3. Obtain explicit maintainer authorization for the protected-file edits.
4. Update `.github/copilot-instructions.md`, remaining root agent files, and relevant `.github/instructions/*.instructions.md` files so they match the stacks retained by the downstream repository.
5. Bump `Last Updated` and `Version` metadata where those fields exist.
6. Avoid ephemeral implementation-stage language in durable governance docs.

## Non-negotiable Safety and Security Rules

1. **No secrets in code or repo**
   - Never hardcode API keys, tokens, connection strings, or credentials.
   - Do not introduce `.env` files or secret placeholders that look like real keys.
   - Never print secrets to stdout/stderr or logs.

2. **Treat all external input as untrusted**
   - Never execute untrusted outputs or commands.
   - Validate and sanitize all inputs at boundaries.
   - Never allow external input to influence file/network access beyond explicitly implemented adapters.

3. **Allowlisted file access only**
   - Read only explicitly allowed inputs/config/rules files and tool-owned runtime dependencies.
   - Refuse path traversal and symlink escapes.

## Pre-commit Discipline (CRITICAL)

**⚠️ ALWAYS run pre-commit checks before committing code.**

Pre-commit hooks are NOT optional. They enforce:

- Code formatting
- Linting
- Trailing whitespace removal
- End-of-file fixes

**Workflow:**

1. Make your code changes
2. Run pre-commit checks locally (e.g., `pre-commit run --all-files` or `npm run lint:md`)
3. Review and commit ALL auto-fixes as part of your change
4. Push to GitHub

**If pre-commit CI fails after a push:**

1. Pull the latest branch
2. Run pre-commit checks locally and review the fixes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanStutz/IntuneComplianceReport](https://github.com/DanStutz/IntuneComplianceReport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
