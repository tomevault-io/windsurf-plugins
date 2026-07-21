---
trigger: always_on
description: See [CONTRIBUTING.AI.md](https://github.com/scratchfoundation/.github/blob/main/CONTRIBUTING.AI.md) for Scratch's
---

# Agent Guide: scratch-blocks

## AI-assisted development policy

See [CONTRIBUTING.AI.md](https://github.com/scratchfoundation/.github/blob/main/CONTRIBUTING.AI.md) for Scratch's
org-wide policy on AI-assisted contributions. The short version: human developers remain responsible for all code
they submit. Do not submit code you cannot explain and defend in a review.

## Agent defaults

Use these defaults unless the user asks otherwise:

1. Keep changes minimal and scoped to the user request. Do not refactor surrounding code, add features, or clean up
   style in areas you weren't asked to touch.
2. Do not preserve backward compatibility when it isn't required. When all callers are internal to the repo, rename
   or restructure freely rather than adding shims or aliases. scratch-blocks is published to npm and consumed
   externally, so treat its public exports as a contract and preserve compatibility unless explicitly told otherwise.
3. Write comments that explain the current code, not its history. Do not reference prior implementations,
   intermediate states, or what the code "used to do." If an approach seems counterintuitive, explain why it is
   correct now — not why it changed.
4. Never edit `node_modules/blockly/`; extend or override from `src/` instead.
5. Prefer fixing root causes over adding surface-level workarounds or assertions.
6. When fixing a bug, start by adding one or more failing tests that reproduce it, then implement the fix. Iterate
   until all tests pass, including but not limited to the new tests.
7. When adding runtime guards for states that should never happen, log actionable context (function name, relevant
   IDs, key flags) rather than failing silently. Use `console.warn` for recoverable states and `console.error` for
   invalid required data.
8. Preserve failure semantics when refactoring. An implicit crash (null dereference, `!` assertion) should become
   an explicit `throw` with a useful message — not silent failure. Code that previously wouldn't crash still
   shouldn't, but consider whether a warning is warranted. Replacing a potential null dereference with
   `if (!foo) return` could make a bug harder to find; `if (!foo) throw new Error(...)` surfaces it.
9. Do not add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and
   framework guarantees. Only validate at system boundaries (user input, external APIs).

## What this repository is

`scratch-blocks` is a TypeScript library that provides the visual block editor for Scratch. It is **not** a fork of
Blockly — it is a library that **depends on** Blockly as an npm dependency and extends it with Scratch-specific block
definitions, fields, rendering, events, UI components, and variable/procedure management.

The compiled output (`dist/main.mjs`) is consumed by components in the `scratch-editor` mono-repo to render the block
palette and workspace.

## Build and lint

```sh
npm run build        # Compile TypeScript and bundle with webpack → dist/main.mjs
npm run format       # Auto-format code
npm run test:lint    # Check for lint and formatting issues (does not run other tests)
npm run test         # Run unit and browser tests (but not lint)
npm run test:unit    # Run unit tests only
npm run test:browser # Run browser tests only
```

Run `npm run test:lint` first when iterating — it is fast. Run `npm run test` before declaring work done.

## Repository layout

```text
src/
├── blocks/        Block definitions for each Scratch category
├── events/        Custom Blockly event subclasses
├── fields/        Custom Blockly field subclasses
├── renderer/      Custom renderer (ScratchRenderer) + cat blocks variant
└── index.ts       Entry point; registers everything with Blockly
```

Key top-level files: `procedures.ts`, `variables.ts`, `data_category.ts`,
`scratch_continuous_toolbox.ts`, `checkable_continuous_flyout.ts`, `scratch_comment_bubble.ts`.

## Blockly is a read-only dependency

The scratch-blocks codebase extends Blockly, but we do not control the Blockly source. Treat files under
`node_modules/blockly/` as read-only.

- If your change can be implemented by extending or overriding Blockly without modifying it, do that.
- If a Blockly change seems necessary, flag it to the human reviewer so they can decide whether to open a Blockly
  issue or PR.

## Terminology

Blockly and Scratch have some overlapping but not identical terminology. These terms sometimes collide in Scratch
Blocks, so here are some definitions to clarify:

- Blockly's "Theme" matches up with Scratch's "Color Mode" — it defines block/category colors and can also affect
  styling of other Blockly UI components. Examples of Scratch's color modes include "default" and "high contrast".
- Scratch's "Theme" is an orthogonal concept. Examples of Scratch's themes include "classic" and "cat blocks". To
  disambiguate this from Blockly themes, we refer to this with `scratchTheme` in the `scratch-blocks` codebase.
- Scratch's block workspace or scripting area is the main Blockly workspace, but it isn't the only Blockly workspace.
  Try to avoid using Blockly's `getMainWorkspace()` method, and instead try to retrieve or pass around a workspace

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scratchfoundation/scratch-blocks](https://github.com/scratchfoundation/scratch-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
