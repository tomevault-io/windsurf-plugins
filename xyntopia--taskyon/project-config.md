---
trigger: always_on
description: - `development_instructions.md` — typing, lint, and root-cause rules (mandatory).
---

# Agent Instructions

## Required reading before code changes

- `development_instructions.md` — typing, lint, and root-cause rules (mandatory).
- `packages/modelica/README.md` — before any Modelica compiler, template, runtime, or simulation change.
- `packages/rumoca/AGENTS.md` — before any rumoca change (separate submodule with own spec system).

## Key rules

- Fix root causes: keep values strongly typed at their source. No `as any`, `as unknown as`, JSON round-trip hacks, or widening types to silence errors.
- If a fix starts cascading into broad type churn, readonly workarounds, or many unrelated file edits, stop and step back. Revert the speculative path and choose the smallest boundary fix instead of spreading the workaround through the codebase.
- No generic `isRecord`-style guards. Narrow at the domain boundary, then pass typed values downstream.
- No fake no-op implementations. Model unavailable capabilities as optional.
- Before creating or refactoring a tool, first search for similar tools in `packages/taskyon/src/tools/` and inspect how they are implemented. Reuse local patterns like `ctx.createSubtasksResult`, `toolCall`, `chatCompletion`, and re-entry chains instead of inventing a new orchestration style.
- Taskyon tool declarations should keep the tool object readable in one place. Prefer one explicit `createTool({ ... })` declaration with the name, description, parameters schema, render options, and `function` body together. Do not split a tool into a wrapper factory, separate parameter constant, or delegated `function: (...) => runSomeTool(...)` unless that abstraction is reused by multiple tools or removes real complexity.
- Tool parameter schemas are the source of truth for tool settings. Do not export or import separate tool-specific settings schemas for UI shortcuts. UI settings views should read schemas from runtime tool definitions, the same way execution reads tool defaults from `tool.parameters`.
- If a `createTool` function body grows too large, extract detail logic into named helper functions, but keep the top-level workflow visible inside the inline `function` body. For example, routing logic such as an entry-node `match(...)` should stay in the tool function so a developer can understand the tool flow in one place.
- When a Taskyon tool returns workflow composition, make the returned task chain explicit at the call site. Prefer visible arrays such as `[task1, task2, task3]` or `[[...branchA], [...branchB]]` near `createSubtasksResult(...)`. Do not hide the number, order, or branching shape of returned tasks behind thin helpers.
- Diagnostics and Taskyon test modules must stay compatible with both browser diagnostics and Node/`tycli` diagnostics. Do not add tests that only work in one runtime unless the unsupported runtime is explicitly modeled and skipped at the diagnostics boundary.
- Diagnostics that exercise LLM behavior must reuse the active runtime/profile settings from the harness (browser profile, `tycli` config, or explicit diagnostic overrides). Do not construct parallel provider/model/API config inside individual tests; if a standard baseline is needed, reset or select that profile before running the diagnostic.
- Do not auto-run `yarn lint` or `yarn lint:fix` (neither repo-wide nor targeted) unless the user explicitly asks. Linting is intentionally not default because it is comparatively expensive. Instead, remind the user to run `yarn lint` themselves before committing, or ask whether they want you to run it when wrapping up. Targeted `yarn eslint <path>` is fine only when needed to verify a specific change and only when explicitly requested.
- Always run the formatter on every file you edited yourself, without waiting for the user to ask. Format only the edited files: `yarn format:file <path...>`
- When adding a test, make sure it is part of the diagnostics suite. Prefer locations already discovered by `packages/taskyon-headless` (for example `packages/taskyon/src/tests/test*.ts`) or wire the new test into the appropriate diagnostics runner.
- Prefer explicit event/function flow over Vue watchers. Watchers are hard to trace and should be used only when reacting to external reactive state is genuinely the simplest boundary; do not use a watcher to bounce one source of truth into another.

## Critical evaluation

- Be more critical of user requests than feels comfortable. Do not treat the requested implementation shape as correct just because the user suggested it.
- Before adding a helper, abstraction, schema, wrapper, or new file, actively look for the existing upstream boundary that should own the behavior. Prefer extending that boundary over creating a parallel path.
- If the user suggests something that duplicates existing logic, weakens a source-of-truth boundary, or adds coordination state, push back clearly and propose the smaller/root-cause alternative.
- Ask whether a change is really needed when the codebase already has an idiomatic mechanism. Agreement is not useful unless the request survives that check.

## TypeScript readability

- Optimize TypeScript for local readability first, then reuse. Strong types are
  required, but do not split every tiny local concept into top-level aliases just

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xyntopia/taskyon](https://github.com/Xyntopia/taskyon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
