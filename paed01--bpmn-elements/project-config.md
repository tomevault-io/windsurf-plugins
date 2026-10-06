---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, and any tool that reads `AGENTS.md`) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, and any tool that reads `AGENTS.md`) when working with code in this repository.

## Workflow

- **TDD is the default.** Red → green → refactor: write or adjust a failing test before changing implementation. Don't delete or weaken existing assertions to land a change — extend them.
- **Performance and coverage are the project's USP.** Avoid regressions in either. On hot paths (broker dispatch, flow traversal, activity activation, joins, multi-instance loops), prefer existing `Context` Maps/refs over rebuilt scans, and avoid per-message allocations/closures where they can be hoisted.
- **JSDoc is concise.** Short intent descriptions are fine; never describe internal implementation.
- **Implementation is never contorted for the type checker.** Types exist to help a consuming dev; `src/` JSDoc documents the public surface and feeds the generated `types/index.d.ts`. When tsc complains about implementation code, put a `// @ts-ignore` line above the statement — never an inline `/** @type {...} */ (expr)` cast, and never restructure working code to satisfy types. The type gate is `npm run typecheck` (part of lint): the root project validates declarations, the `test/` project (checkJs, `test/tsconfig.json`) type-checks the suite as the consuming dev — test files use JSDoc annotations and casts freely. Tests import from `bpmn-elements` (and its subpaths) so they exercise the shipped `types/index.d.ts`; only internals the package does not export (`EventBroker`, `Api`, the execution orchestrators, …) are imported via `src/` paths, with `/** @type {any} */` casts bridging the two type flavors at those seams.
- Before declaring done: `npm test` (full suite + lint + `dist` rebuild). For coverage-sensitive work, also `npm run cov:html`.

## Commands

- `npm test` — run the full suite in parallel (mocha, `mocha-cakes-2` UI, hot-bev reporter, 3000ms timeout). `posttest` then runs lint and rebuilds `dist/`.
- `npm run lint` — `eslint . --cache && prettier . --check --cache`.
- `npm run dist` — Babel transpile `src/` → `dist/` (also runs on `prepack`).
- `npm run cov:html` — c8 HTML coverage report.
- `npm run test:md` — run texample against code blocks in the documentation markdown files.
- Single test file: `npx mocha test/feature/activity-feature.js`. `.mocharc.json` auto-loads `mocha-cakes-2` and `test/helpers/setup.js` (which registers chai `expect` globally and sets `NODE_ENV=test`).
- Single scenario: add `-g "scenario name"` to the mocha invocation above.
- Note: default mocharc timeout is 1000ms; the `npm test` script bumps it to 3000ms. Long-running scenarios may need `-t 3000` when run standalone.

## Architecture

The library executes BPMN 2.0 workflows. The execution model is message-driven — almost nothing happens by direct method call — so this section focuses on what you cannot learn from any single file.

### Execution hierarchy: Definition → Process → Activity

Each layer pairs a structural wrapper with a dedicated execution orchestrator:

- `src/definition/Definition.js` + `src/definition/DefinitionExecution.js` — top-level, manages executable processes and inter-process messaging.
- `src/process/Process.js` + `src/process/ProcessExecution.js` — owns one `<bpmn:process>`, handles flow traversal, joins, and parallel activation.
- `src/activity/Activity.js` + `src/activity/ActivityExecution.js` — wraps any element (task, event, gateway), tracks postponed/waiting state, drives a per-run behavior instance.

### Message-driven core via `smqp`

All coordination is async message passing on an in-memory AMQP-like broker (`smqp`, a runtime dependency). Each element owns its own `EventBroker` (`src/EventBroker.js`) with exchanges named `event`, `run`, `format`, `execution`, and `api`. Per-element factories wire these up: `DefinitionBroker`, `ProcessBroker`, `ActivityBroker`, `MessageFlowBroker`.

Execution is driven by publishing routing keys like `execute.start`, `execute.completed`, `execute.error`, `run.enter`, `run.end`, `run.discard`, and subscribing via `broker.subscribeTmp()` / `subscribeOnce()`. Messages with `mandatory: true` surface errors if undelivered. The `EventBroker` exposes convenience methods: `on`, `once`, `waitFor`, `emit`, `emitFatal`. If you try to read `ActivityExecution` or `ProcessExecution` as imperative code you will get lost — keep the publish/subscribe model in mind.

**Do not read synchronous queue/exchange state off the broker** (`queue.messageCount`, `consumerCount`, `peek`, etc.). These are `smqp` conveniences a host that swaps in a real AMQP-compliant broker will not provide. Track what you need in execution state instead — e.g. `consumeInbound` keys the consumer assertion off the `initialized` counter, not a pending-message count on `inbound-q`. Publishing, subscribing, asserting/cancelling consumers, and `queueMessage` are all fine; reading queue depth is not.

### Activity vs Behaviour

An element type like `ServiceTask` is not a class. It is a factory function that returns an `Activity` constructed with a `Behaviour` class:

- `Activity` holds structural info: id, type, inbound/outbound flows, broker, lifecycle state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paed01/bpmn-elements](https://github.com/paed01/bpmn-elements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
