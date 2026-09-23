---
trigger: always_on
description: A DeepSeek Harness plugin for context insight, actions, and management.
---

# dsh-context

A DeepSeek Harness plugin for context insight, actions, and management.

## Background

- DeepSeek Harness (dsh):
  - an open-source agent harness developed by DeepSeek AI.
  - Github: https://github.com/deepseek-ai/deepseek-harness
  - NPM: @deepseek-ai/dsh
  - MUST ensure the full clone of the DeepSeek Harness repository is available locally, before any work.
  - MUST always dive deep into the details of dsh source code and dependencies, for its mechanisms, lifecycles, and modules. Ensure every decision is based on the full and actual truth of the dsh source code.
  - Local git clone of [dsh](https://github.com/deepseek-ai/deepseek-harness):
    - may be found in the `~/dev/deepseek-harness` directory
    - `git pull` on the `main` branch to update
    - commits and version tags are available for reference and diff
    - run `pnpm install` to update dependencies after a `git pull` or switching commit/tag

- DeepSeek Harness Plugin:
  - docs:
    - Reference: https://deepseek-harness.github.io/deepseek-harness/en/reference/
  - Example plugins:
    - Available on GitHub topic `dsh-plugin`: https://github.com/topics/dsh-plugin

## Coding
- Always consider the minimal change and the most performance efficient implementation.
- Try best to use the existing classes, utilities, styles, style tokens, events, presets and lifecycles provided by DeepSeek Harness.ess.
- Use English in code comments, documentation, Pull Request description, and commit messages.
- Smaller, less-coupling and modulized code and tests are preferred for better maintainability and testability.
- Avoid adding unnecessary code comments (unless for the pinned major decision or for those provide significant value) and code duplication.
- Update or remove the outdated or unhelpful code comments when modifying the code.
- Before any commit, MUST ALWAYS do ALL the following checks:
  - Check the to-do list, and ensure all the items are properly completed or closed.
  - Carefully independently review and simplify all the diffs and all code changes, to ensure they are necessary, correct and not over-engineered. 
  - Cleanup the generated temporary files. Cleanup temporary or unhelpful comments.
  - MUST Run `pnpm run lint:fix && pnpm run test && pnpm run build` in single command and capture FULL output, to ensure:
    - passing all the linting and test
    - the per-file code coverage MUST BE literally 100%.
      - The coverage table lists ONLY the files below 100% (`coverage.skipFull` in `vitest.config.ts`): a passing run prints an empty table (headers only, no `All files` row), and the run also fails the `coverage.thresholds` gate when any file drops below 100 — the offending files then appear in the table.
      - Example passing output (nothing below 100%):
        - % Coverage report from v8
          -------------------|---------|----------|---------|---------|-------------------
          File               | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
          -------------------|---------|----------|---------|---------|-------------------
          -------------------|---------|----------|---------|---------|-------------------

## Parsing resilience (log data must never crash or hang a view)

The plugin lives off data it does not own: the durable session log (event shapes vary across dsh versions, producers, and hand-edited replays), the conversation snapshot behind the client join, projection payloads on the wire, history RPC pages, and persisted stores. Treat all of it as untrusted input at every layer. The two failures to design out: any client- or host-side parsing that blanks the page (the error card), and anything that leaves the page stuck on "loading".

- Never let one bad record take down a view. A malformed node, event, tool entry, or file op degrades to zero rows for that item — the card, the tab, and the session keep working.
- Host-side projection folds must be TOTAL. The harness projection registry drives `apply` straight off the session/event bus with no error boundary of its own: one throwing fold stalls that unit's cells and its `session/projection` push feed, and the browser then waits on "loading" forever. So: unknown event types return the state unchanged; per-event processing is isolated so a malformed event is dropped whole (all-or-nothing — no partial state); and never materialize an `undefined`-valued property into persisted state, because the plain-JSON precondition makes one such property fail EVERY projection-cache write for the session (sessions then break in unrelated, far-away places).
- Client-side parsing degrades visibly. Sanitize delivered projection payloads at the boundary (the `timelineOf` pattern: collections re-proved, scalars zeroed, whole-value absence stays `null` → loading screen); isolate per-item work in any fold over join/log data (per-item guards, or a bounded catch when a hostile object may throw on property access); every async fetch must resolve to data or a visible retryable state, never an unhandled rejection that leaves a spinner.
- Re-prove every field at runtime. Structural narrowing over blind casts; optional chaining over non-null assertions; skip elements that fail the shape instead of throwing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bowenliang123/dsh-context](https://github.com/bowenliang123/dsh-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
