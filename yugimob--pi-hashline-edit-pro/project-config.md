---
trigger: always_on
description: A fork of [pi-hashline-edit](https://github.com/RimuruW/pi-hashline-edit) (MIT) that preserves the strict semantics of the original and makes two compounding changes:
---

# Repository Guidelines

## What this is

A fork of [pi-hashline-edit](https://github.com/RimuruW/pi-hashline-edit) (MIT) that preserves the strict semantics of the original and makes two compounding changes:

1. **Hash format: 3 characters with perfect hashing.** Combined with the alphabet change below, this gives 18 bits of entropy per anchor (262,144 buckets), up from 8 bits (256 buckets) in upstream. The 3-character anchors use collision resolution: if a computed hash collides with an already-assigned hash, it increments until a unique hash is found. This ensures every line in a file gets a unique anchor.
2. **Alphabet: 16-char hand-curated → 64-char URL-safe base64.** The original alphabet excluded hex digits, vowels, and visually confusable letters to be friendly to a human reader. The consumer here is an LLM that tokenizes, not a human that squints at pixel glyphs — so the human-readability heuristics don't apply and the full 64 chars give max entropy per position.

The strict-semantics policy of the original is preserved verbatim. This fork is a parameter change, not a philosophy change.

## Project Structure & Module Organization

- `index.ts` is the extension entrypoint; it registers the custom `read`/`replace` tools, disables the built-in `edit` tool, and registers the `/toggle-auto-read` command. It also contains the auto-read-after-write handler (disabled by default; controlled by `PI_HASHLINE_AUTO_READ` env var or the `/toggle-auto-read` command).
- `src/` contains the implementation, split by responsibility: `read.ts`, `replace.ts`, `replace-normalize.ts`, `replace-diff.ts`, `replace-response.ts`, `replace-render.ts`, `file-kind.ts`, `fs-write.ts`, `snapshot.ts`, `utils.ts`, and small runtime/path helpers. The hashline engine is in `src/hashline/` with sub-modules: `hash.ts`, `parse.ts`, `resolve.ts`, `apply.ts`, and `index.ts` (re-exports).
- `prompts/` holds the Markdown prompt text loaded by the tools at runtime.
- `test/` mirrors the code layout: `core/` for hashline primitives, `tools/` for tool behavior, `extension/` for registration, `integration/` for end-to-end flows, and `support/fixtures.ts` for temp-file helpers.
- `assets/` is documentation media only.

## Build, Test, & Development Commands

- `npm install` — install dependencies.
- `npm test` — run the full test suite with `vitest`.
- `npm test -- test/tools` — run tool-facing tests while iterating on `read`/`replace` behavior.
- `npm test -- test/integration/strict-hashline-loop.test.ts` — run the strict hashline integration scenario.
- There is no separate build step today; Pi loads the TypeScript entrypoints directly from `index.ts`.

## Coding Style & Naming Conventions

- Use TypeScript with ESM imports, two-space indentation, double quotes, and semicolons to match the existing codebase.
- Keep modules narrow and named by responsibility (`fs-write.ts`, `replace-normalize.ts`).
- Export typed functions and use specific error paths; avoid broad refactors or speculative abstractions.
- No ESLint or Prettier config is checked in, so preserve local style and keep diffs tight.

## Testing Guidelines

- Write tests with `vitest` and place them under the matching `test/` subfolder.
- Name files `<feature>.test.ts`; group assertions around one behavior per `describe` block.
- Any change to anchor parsing, diff preview, request normalization, or atomic writes should include or update tests in the affected layer.
- New integration scenarios (e.g. compound edits, stale-position edge cases) go under `test/integration/` as standalone `<scenario>.test.ts` files.

## Commit & Pull Request Guidelines

- Follow the existing Conventional Commit pattern: `fix(hashline): ...`, `refactor(read, edit): ...`, `docs: ...`.
- Keep commits focused and imperative; separate behavior changes from documentation-only changes.
- PRs should summarize the user-visible effect, list the tests run, and include before/after snippets when tool output or prompts change.

## Architecture Guardrails

- Keep `read`, `replace`, prompt text, and tests in sync whenever the hashline format changes.
- Do not bypass `src/fs-write.ts`; atomic writes are part of the extension's safety guarantees.
- Preserve stale-anchor rejection semantics unless the change explicitly redesigns the protocol.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuGiMob/pi-hashline-edit-pro](https://github.com/YuGiMob/pi-hashline-edit-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
