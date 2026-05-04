---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- OPENSPEC:START -->
## OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

## Planning
When planning or creating specs, use AskUserQuestions to ensure you align with the user before creating full planning files.

Also, please add a 'What's Next' section at the end of each plan. This will let us chain plans and clear context in a smart manner.

## Instruction File Parity

`CLAUDE.md` and `AGENTS.md` are required to stay identical in this repo.

- Pre-commit hook `sync-agent-instructions` auto-syncs them.
- If both files are edited differently in one change, resolve manually so they match exactly.

## Debugging Assumptions

**IMPORTANT**: When the user asks you to check logs from a MassGen run, assume they ran with the current uncommitted changes unless they explicitly say otherwise. Do NOT assume "the run used an older commit" just because the execution_metadata.yaml shows a different git commit - the user likely ran with local modifications after you suggested changes. Always debug the actual code behavior first.

## Implementation Guidelines

After implementing any feature that involves passing parameters through multiple layers (e.g., backend -> manager -> component), always verify the full wiring chain end-to-end by tracing the parameter from its origin to its final usage site. Do not rely solely on unit tests passing -- add an integration smoke test or assertion that the parameter actually arrives at its destination, not just that the downstream logic works when the parameter is provided.

For cross-backend tool-calling behavior (OpenAI-compatible, Claude, Gemini, Codex/Claude Code via MCP, etc.), treat backend-side argument normalization and schema validation as the source of truth. Prompt guidance can encourage correct argument shape, but correctness must not depend on prompt compliance alone (e.g., tolerate/detect accidentally stringified JSON argument payloads in adapters and tool gateways).

When fixing `E501` in prompt text (especially triple-quoted system prompts), preserve the rendered prompt exactly. Wrap long source lines using escaped line continuation (`\` at end-of-line) so lint passes without introducing extra newline characters or changing prompt behavior.

## Anti-Patterns

- **No keyword/heuristic matching for categorization or similarity.** Avoid writing code that infers categories, detects similarity, or organizes content using keyword lists, Jaccard similarity, Levenshtein distance, or similar heuristics. These approaches are brittle and produce low-quality results. Use LLM-based approaches instead -- that's the whole point of this project.
- **No explicit tool call syntax in prompts.** Do not hardcode specific tool/function call syntax (e.g., `tool_name(param="value")`) in system prompts, skill instructions, or user-facing text. Models determine the correct calling convention from tool schemas. Describe what the agent should do in natural language instead. Hardcoded syntax is fragile across providers and couples prompts to implementation details.

## Test-Driven Development (TDD)

**TDD is the default development methodology for this project.** With 121 test files and 1580+ tests across unit, integration, frontend, and snapshot layers, the test infrastructure is mature. All non-trivial work MUST follow the TDD cycle.

### The TDD Contract

For every non-trivial change (bug fix, new feature, refactor, backend change, TUI work, WebUI work), follow this sequence **in order**:

1. **Agree on acceptance tests first.** Before writing any production code, align with the user on what tests will prove the change works. Define pass/fail criteria explicitly.
2. **Write the tests.** Implement or update tests that express the desired behavior. These tests MUST fail initially -- if they pass before you write production code, either the tests are wrong or the feature already exists.
3. **Confirm tests fail for the right reason.** Run the tests and verify they fail because the feature/fix is missing, not because of a test bug.
4. **Implement the minimum code to pass.** Write production code until the test suite passes. Do not add untested behavior.
5. **Refactor under green.** Clean up only while all tests remain green.
6. **Commit tests alongside code.** Tests are not throwaway scaffolding -- they are permanent regression protection.

### When to Apply TDD

| Change Type | TDD Required? | Rationale |
|-------------|:---:|-----------|
| New feature / capability | **Yes** | Define behavior before implementing it |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [massgen/MassGen](https://github.com/massgen/MassGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
