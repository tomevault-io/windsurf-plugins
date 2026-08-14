---
trigger: always_on
description: - Keep language consistent and understandable when directing Codex.
---

# Codex Collaboration Guide

## Purpose
- Keep language consistent and understandable when directing Codex.
- Provide the minimum context Codex needs to work safely and efficiently.
- Ensure every change is validated before it is shared or released.

## Repository Quick Map
- `src/`: VS Code extension client (commands, UI helpers, activation logic).
- `packages/pml-language-server/src/`: TypeScript language server (parser, diagnostics, providers).
- `examples/`: Sample PML files used for manual testing.
- `out/`: Generated build artefacts (never edit manually).
- `assets/`, `icons/`, `snippets/`, `syntaxes/`: User-facing resources; keep filenames and formats stable.

## Coding Standards & Constraints
- TypeScript only; follow existing module structure and prefer pure functions.
- Reuse helper utilities (`packages/pml-language-server/src/utils`) before adding new ones.
- Comments should explain intent, not restate code. Avoid adding Russian text—use English throughout.
- Do not add runtime dependencies without prior approval; devDependencies require justification.
- Maintain ASCII unless the file already uses Unicode characters for good reason.
- Keep diffs focused: group related changes, avoid drive-by refactors.

## Testing & Validation
- Bundled compile: `npm run compile` (builds the extension bundle).
- TypeScript baseline: `npm run compile:tsc` (runs root and language-server TypeScript builds).
- Language server tests: `npm --prefix packages/pml-language-server run test` (Vitest).
- Packaging smoke test (when functionality changes): `npm run pack` and install the generated VSIX in a disposable VS Code profile.
- Record test commands and outcomes in the task notes; surface failures instead of hiding them.

## Implementation Workflow
1. Clarify the requirement (issue, user request, or roadmap item). Capture edge cases before coding.
2. Inspect relevant files to understand current behaviour; note constraints or TODO comments.
3. Make incremental commits on a feature branch; keep commit messages action-oriented.
4. After each logical change, run the appropriate tests/compiles. If something fails, stop and document the failure with logs.
5. Before requesting review or merging, scan the diff for unintended changes (formatting, large asset churn, regenerated artefacts).

## Continuous Improvement Loop
- Treat this repository as an active production product: after one task is implemented and validated, immediately identify the next highest-value improvement unless the user explicitly pauses.
- Keep the loop moving in this order: review current state, capture findings, ask an external reviewer such as Claude/Codex for read-only review when useful, compare findings, plan the next focused implementation, make changes, validate, update release notes/roadmap, then select the next item.
- Do not wait for a perfect roadmap before improving the extension. If no explicit task is queued, choose a small, defensible item from stabilization, performance, diagnostics quality, completion quality, navigation, snippets/presets, packaging safety, or test coverage.
- Prefer improvements that reduce customer risk or user friction: fewer false diagnostics, faster indexing/completions, better Outline/navigation, safer commands, clearer settings, better snippets, and stronger release validation.
- Keep each implementation slice small enough to review. Avoid mixing unrelated parser changes, UI changes, release automation, and documentation unless they are required for the same release item.
- After every implementation slice, run targeted tests first, then the broader required validation when the change is release-relevant.
- If validation passes, update `CHANGELOG.md`, `RELEASE_NOTES.md`, and `ROADMAP.md` when the change is user-visible, release-relevant, or changes project direction.
- If validation fails, stop feature work, document the failure, and either fix the failure or revert only the changes from the current slice.
- Do not publish, push tags, or create a GitHub release without the user's explicit `confirm publish`.
- When handing work to another AI reviewer, provide a review-only prompt that forbids edits, commits, pushes, publishing, and fact invention. Treat that reviewer as advisory; verify findings before applying fixes.
- End every final response with the next concrete action or plan. If the work is blocked or the team is in a planning loop, state the blocker, discuss the next plan, and continue with the loop.

## External Reviewer Invocation
- Claude review is a required checkpoint before release prep, before risky UX/parser/diagnostic changes are finalized, and whenever the user asks for external review.
- Prefer the installed Codex slash command `/claude-review`.
- If invoking Claude manually, Codex must collect the necessary diff/context first and pipe that plain text bundle to Claude. Do not ask Claude to inspect the workspace itself.
- Use the local Claude Code subscription flow via the `claude` command. Do not use `ANTHROPIC_API_KEY` for these reviews.
- Invoke Claude non-interactively with no tools:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikhalchankasm/vscode-pml-aveva-e3d](https://github.com/mikhalchankasm/vscode-pml-aveva-e3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
