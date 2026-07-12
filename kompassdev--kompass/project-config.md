---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md - kompass

Guidelines for AI agents working in this repository.

## What This Is

This is a Kompass workspace with multiple packages.

- `packages/core` contains the generic workflow toolkit
- `packages/opencode` contains the OpenCode adapter

Compiled OpenCode artifacts are written to `packages/opencode/.opencode/` for review.

## When Making Changes

```bash
# Run after making code or generated-file changes in this session
bun run compile
bun run typecheck
bun run test
```

- Only run these commands after you edit files in this session.
- If you are only analyzing an existing branch, reviewing changes, or creating a PR without editing files, do not run them automatically.
- Do not regenerate `packages/opencode/.opencode/` unless you changed the source that produces it or the user explicitly asked.
- When you add, remove, or rename commands, agents, components, tools, adapter settings, or bundled config fields, keep every source of truth in sync in the same change:
  - runtime definitions in `packages/core`
  - bundled config in `kompass.jsonc`, `packages/core/kompass.jsonc`, and `packages/opencode/kompass.jsonc`
  - schema in `kompass.schema.json`
  - user-facing docs in `README.md`, adapter/package docs, and the web docs under `packages/web/src/content/docs/` that describe the changed surface
  - web marketing and homepage surfaces that showcase commands or agents, especially `packages/web/src/pages/index.astro` and `packages/web/src/components/CommandShowcase.astro`
  - generated OpenCode output under `packages/opencode/.opencode/` when the source change affects compiled artifacts
- If no validation was run in the current session, say that clearly instead of implying the branch was tested.

Never edit `packages/opencode/.opencode/` directly.

## Project Structure

```text
packages/core/      # Shared commands, agents, components, tools, tests
packages/opencode/  # OpenCode adapter package
kompass.jsonc       # Local workspace config used for development
packages/opencode/.opencode/ # Generated OpenCode output for review
```

## Package Boundaries

- Put reusable workflow logic in `packages/core`
- Put OpenCode-specific SDK wiring in `packages/opencode`
- Do not make the workspace root a runtime package again

## Command Authoring

- Author command definitions in `packages/core/commands/`; treat `packages/opencode/.opencode/commands/` as generated output only
- Treat `packages/core/commands/index.ts`, `packages/core/lib/config.ts`, `kompass.schema.json`, the bundled `kompass.jsonc` files, the relevant docs, and any web showcase/homepage surfaces as a linked surface area; if one changes, verify the others still describe the same command set, agent ownership, examples, and config shape
- Use `packages/core/commands/pr/create.md` as the canonical example for command structure and tone
- Keep this section order in command docs unless a command has a strong reason not to: `## Goal`, `## Additional Context`, `## Workflow`
- Keep `### Output` as the final subsection inside `## Workflow`; do not use a separate top-level `## Output` section
- Start `## Workflow` with a dedicated `### Arguments` subsection that stores the raw `$ARGUMENTS` value inside literal `<arguments>` tags before any normalization
- Follow `### Arguments` with `### Interpret Arguments`, and normalize `<arguments>` into any additional named placeholders before execution steps
- Use angle-bracket placeholders consistently for derived values and stored context, such as `<arguments>`, `<base>`, `<additional-context>`, `<pr-url>`, and define each placeholder before it is referenced later in the command
- When referring to placeholders literally in prose, always wrap them in backticks, such as `<arguments>` or `<pr-url>`; keep output examples plain when the placeholder represents substituted user-facing text
- If arguments can mean different things, explicitly disambiguate them in `### Interpret Arguments` and store each interpretation in a separate placeholder
- For navigator-style commands, separate context loading, blocker checks, delegated execution, and final reporting into distinct workflow subsections so the control flow is easy to follow
- Prefer explicit subsection names like `### Load ... Context`, `### Check Blockers`, `### Delegate ...`, and `### Mark Complete And Loop` when the command coordinates multiple phases or subagents
- Treat loader tools and provided attachments as the source of truth for orchestration inputs; avoid extra exploratory commands when an existing tool result already answers the question
- Before dispatching a same-session command step, say what delegated result should be stored and whether the workflow must stop, pause, or continue based on that result
- Use literal `<delegate>` tags when the workflow must delegate exact text through `command_expansion`; `agent` and `command` are required, and the block body is the exact rendered body to send for that command
- Do not use `<task>` blocks in command docs; author navigator delegation with `<delegate>` blocks only
- Do not restate `command_expansion` or `task` mechanics inside command docs; navigator owns that execution flow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kompassdev/kompass](https://github.com/kompassdev/kompass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
