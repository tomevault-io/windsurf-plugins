---
trigger: always_on
description: > TEMPLATE NOTE: Copy this file to the solution root as `AGENTS.md`, then replace every `TODO:` and every `...` with repo-specific values. Delete this note once the file is customized.
---

# AGENTS.md

> TEMPLATE NOTE: Copy this file to the solution root as `AGENTS.md`, then replace every `TODO:` and every `...` with repo-specific values. Delete this note once the file is customized.

Project: TODO
Stack: TODO

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Purpose

This file defines how AI agents work in this solution.

- Root `AGENTS.md` holds the global workflow, shared commands, cross-cutting rules, and global skill catalog.
- In multi-project solutions, each project or module root MUST have its own local `AGENTS.md`.
- Local `AGENTS.md` files add project-specific entry points, boundaries, commands, risks, and applicable skills.

## Solution Topology

- Solution root: `...`
- Projects or modules with local `AGENTS.md` files:
  - `...`
  - `...`

## Rule Precedence

1. Read the solution-root `AGENTS.md` first.
2. Read the nearest local `AGENTS.md` for the area you will edit.
3. Apply the stricter rule when both files speak to the same topic.
4. Local `AGENTS.md` files may refine or tighten root rules, but they must not silently weaken them.
5. If a local rule needs an exception, document it explicitly in the nearest local `AGENTS.md`, ADR, or feature doc.

## Conversations (Self-Learning)

Learn the user's stable habits, preferences, and corrections. Record durable rules here instead of relying on chat history.

Before doing any non-trivial task, evaluate the latest user message.
If it contains a durable rule, correction, preference, or workflow change, update `AGENTS.md` first.
If it is only task-local scope, do not turn it into a lasting rule.

Update this file when the user gives:

- a repeated correction
- a permanent requirement
- a lasting preference
- a workflow change
- a high-signal frustration that indicates a rule was missed

Extract rules aggressively when the user says things equivalent to:

- "never", "don't", "stop", "avoid"
- "always", "must", "make sure", "should"
- "remember", "keep in mind", "note that"
- "from now on", "going forward"
- "the workflow is", "we do it like this"

Preferences belong in `## Preferences`:

- positive preferences go under `Likes`
- negative preferences go under `Dislikes`
- comparisons should become explicit rules or preferences

Corrections should update an existing rule when possible instead of creating duplicates.

Treat these as strong signals and record them immediately:

- anger, swearing, sarcasm, or explicit frustration
- ALL CAPS, repeated punctuation, or "don't do this again"
- the same mistake happening twice
- the user manually undoing or rejecting a recurring pattern

Do not record:

- one-off instructions for the current task
- temporary exceptions
- requirements that are already captured elsewhere without change

Rule format:

- one instruction per bullet
- place it in the right section
- capture the why, not only the literal wording
- remove obsolete rules when a better one replaces them

## Global Skills

List only the skills this solution actually uses.
Do not paste the whole framework catalog here.

- `<skill-name>` — when agents should use it
- `<skill-name>` — when agents should use it

If the stack is `.NET`, install the needed `.NET` skills from the [Managed Code Skills catalog](https://skills.managed-code.com/).
The usual baseline often includes:

- `mcaf-dotnet`
- `mcaf-dotnet-features`
- `mcaf-testing`
- exactly one of `mcaf-dotnet-xunit`, `mcaf-dotnet-tunit`, or `mcaf-dotnet-mstest`
- `mcaf-dotnet-quality-ci`
- `mcaf-dotnet-complexity`
- `mcaf-solid-maintainability`
- `mcaf-architecture-overview` if the repo keeps a maintained architecture map
- `mcaf-ci-cd`

If the stack is `.NET`, document skill-management rules explicitly:

- `.NET` skills are sourced from `https://skills.managed-code.com/`.
- `mcaf-dotnet` is the entry skill and routes to specialized `.NET` skills.
- Keep exactly one framework skill: `mcaf-dotnet-xunit` or `mcaf-dotnet-tunit` or `mcaf-dotnet-mstest`.
- Add tool-specific `.NET` skills only when the repository actually uses those tools in CI or local verification.
- Keep only `mcaf-*` skills in agent skill directories.
- When upgrading skills, recheck `build`, `test`, `format`, `analyze`, `complexity`, and `coverage` commands against the repo toolchain.

## Rules to Follow (Mandatory)

### Commands

- `build`: `...`
- `test`: `...`
- `format`: `...`
- `analyze`: `...` (delete if not used)
- `complexity`: `...` (delete if not used)
- `coverage`: `...` (delete if not used)

If the stack is `.NET`, also document:

- whether tests run on `VSTest` or `Microsoft.Testing.Platform`
- whether `format` is `dotnet format --verify-no-changes` or a checked-in wrapper over it
- whether coverage uses a VSTest collector, `coverlet.MTP`, or an MSTest SDK extension
- explicit `LangVersion` only when the repo intentionally differs from the SDK default

### Project AGENTS Policy

- Multi-project solutions MUST keep one root `AGENTS.md` plus one local `AGENTS.md` in each project or module root.
- Each local `AGENTS.md` MUST document:
  - project purpose
  - entry points
  - boundaries
  - project-local commands
  - applicable skills
  - local risks or protected areas

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [managedcode/MCAF](https://github.com/managedcode/MCAF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
