---
trigger: always_on
description: Project: dotnet-skills
---

# AGENTS.md

Project: dotnet-skills
Stack: .NET 10, GitHub Actions, Python automation, NuGet, GitHub Releases

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Purpose

This file defines how AI agents work in this repository.

- Root `AGENTS.md` holds the global workflow, repository structure, release and automation policy, and skill-catalog maintenance rules.
- This repository currently uses only the root `AGENTS.md`; add a nearer local `AGENTS.md` only when a subtree needs stricter or more specialized rules.
- The repository has three equally important responsibilities:
  1. Maintain a high-quality `skills/` catalog for modern and legacy `.NET`.
  2. Maintain repo-owned orchestration `agents/` that route broader tasks into the right skills or narrower agents.
  3. Maintain automation that watches official upstream releases and documentation so the catalog can be refreshed when the ecosystem changes.

If this repository contains executable code, it must exist only to distribute or install the skill catalog itself, for example as a publishable `dotnet tool`.
Do not turn this repository into a general application or unrelated tooling codebase.

All repository-facing documentation and skill content should be written in English.
Follow official or documented agent standards where they exist; do not present a repo-local adapter as if it were a universal standard.

## Solution Topology

- Solution root: `.`
- Areas with specialized responsibilities:
  - `agents/`: top-level orchestration agents that sit above the skill catalog, one folder per agent
  - `skills/`: canonical skill catalog
  - `tools/ManagedCode.DotnetSkills/`: publishable `dotnet-skills` installer tool
  - `scripts/`: catalog generation and upstream-watch automation
  - `.github/workflows/`: CI, release, and scheduled automation
- Local `AGENTS.md` files currently present: none

## Rule Precedence

1. Read the root `AGENTS.md` first.
2. If a local `AGENTS.md` is later added for a subtree, read the nearest one before editing that area.
3. Apply the stricter rule when both files speak to the same topic.
4. Local `AGENTS.md` files may refine or tighten root rules, but they must not silently weaken them.
5. If a subtree needs a durable exception, document it explicitly in the nearest local `AGENTS.md` or another canonical repo document.

## Path And Linking Rules

- Never commit personal or machine-specific absolute filesystem paths such as `/Users/...`, `/home/...`, or `C:\Users\...` in repository docs, generated site files, manifests, examples, or contributor guidance.
- In repository-facing Markdown, prefer repo-relative links such as `README.md`, `skills/`, or `.github/workflows/publish-catalog.yml` instead of workstation-local absolute paths.
- For path examples, use portable placeholders such as `~/...`, `/path/to/...`, `<repo-root>/...`, or product-native paths that are not tied to one contributor machine.
- Before committing docs or generated artifacts, scan the diff for leaked local paths and remove them.

## Conversations (Self-Learning)

Learn the user's stable habits, preferences, and corrections. Record durable rules here instead of relying on chat history.

Before doing non-trivial work, evaluate the latest user message.
If it contains a durable rule, correction, preference, or workflow change, update `AGENTS.md` first.
If it is only task-local scope, do not turn it into a lasting rule.

Update this file when the user gives:

- a repeated correction
- a permanent requirement
- a lasting preference
- a workflow change
- a high-signal frustration that indicates a rule was missed
- a rule request for library skill quality, such as “when adding a library skill, include installation and practical usage patterns in the skill body”

Treat explicit frustration, swearing, sarcasm, repeated rejection, or "don't do this again" as strong signals that a durable rule should likely be captured here.

### Issue Workflow

- When repository work is driven by GitHub issues, complete the implementation end-to-end: inspect the issue, make the repo changes, validate them, commit, push, and close the resolved issues.
- When committing work that resolves repository issues, include the issue-closing references in the commit body, for example `Closes #48`.

Do not record:

- one-off instructions for the current task
- temporary exceptions
- requirements that are already captured elsewhere without change

## Library Skill Standard

- When a user asks to add a skill for a library, update the skill with source-driven usage guidance, not a placeholder.
- The skill must include, at minimum:
  - install path for the library (NuGet/PackageReference/cmd examples where relevant),
  - at least two practical usage snippets (read + write),
  - option/setting patterns that affect behavior,
  - tradeoffs and constraints (for example ref-struct and async limits),
  - validation checks the user can run locally.
- Prefer a practical “how to use this from today” structure: install, read, write, validate.

### Provided Source Links

- When a user provides one or more source URLs for a skill, agent, or durable documentation task, inspect those URLs directly before editing the repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Postpartum-genushyacinthus29/dotnet-skills](https://github.com/Postpartum-genushyacinthus29/dotnet-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
