---
trigger: always_on
description: This guide governs `veryfront-code/`. Deeper `AGENTS.md` files override it for their subtrees. The `cli/` subtree has its own guide at `cli/AGENTS.md` and that guide is the narrower authority for CLI command work.
---

# Veryfront code agent guide

This guide governs `veryfront-code/`. Deeper `AGENTS.md` files override it for their subtrees. The `cli/` subtree has its own guide at `cli/AGENTS.md` and that guide is the narrower authority for CLI command work.

Veryfront Code is the Deno-first framework and runtime package for Veryfront. It contains the public `veryfront/*` APIs, the runtime server, SSR/RSC support, agent and workflow primitives, runs, tasks, MCP support, and CLI entrypoints.

## Default working style

- Prefer the smallest viable diff.
- Preserve public API compatibility unless the task explicitly asks for a breaking change.
- For behavior changes, add or update a focused failing test before changing implementation.
- Reuse existing modules, schemas, adapters, and error patterns before adding new abstractions.
- Keep public behavior grounded in code evidence, generated types, and tests.
- Update docs, examples, generated references, and command help when public behavior changes.
- Do not rename, move, or refactor unrelated code.
- Do not add dependencies unless the task explicitly requires them.

## Commands

Use Deno commands from this repository root.

```bash
# Command discovery
veryfront schema --json

# MCP server
veryfront mcp

# Run the full test suite
deno task test

# Unit tests only, parallel, excluding integration suites
deno task test:unit

# A single file or directory
deno task test:file src/workflow/executor/dag/index.test.ts
```

For targeted changes, run the narrowest relevant task first (`deno task test:file <path>`), then
broaden only when the change touches shared runtime, public APIs, or cross-cutting behavior.

This repository pins its Deno version in `.tool-versions`, and `mise` or `asdf`
will select it automatically. Match it before regenerating anything: generated
files embed declaration line numbers in padded columns, so a different Deno
rewrites them even when nothing changed, and committing that output turns CI red
for everyone. `deno task docs` refuses to run on the wrong version rather than
producing a plausible-looking diff.

Use these tasks rather than a hand-written `deno test` command; they deny network access to
the LLM provider origins.

To control outbound HTTP in a test, use `src/testing/mock-fetch.ts` -- `withMockFetch(mock, fn)`
where the stub has a callback to scope, or the `installMockFetch` / `restoreMockFetch` pair for
suites that install per test and tear down in `afterEach`. Both move `globalThis.fetch`, the
host transport in `src/security/http/outbound-fetch.ts`, and the egress guard's host resolver
together, so a stubbed request performs no DNS at all. Supplying only a transport is not enough:
the guard resolves the destination before any transport sees the request.

Assigning `globalThis.fetch` by hand controls only code that calls `fetch` directly. Anything
routed through `guardedOutboundFetch` reads the host transport instead, so a hand-assigned stub
is ignored there and the request reaches the live endpoint, failing with a confusing 401 or 405.

## Public copy rules

Apply these rules to CLI output, command help, docs generated from this package, examples, error messages, warnings, logs exposed to users, API descriptions, and public comments.

- Use direct, concise language.
- Address the reader as "you". Use "Veryfront" for the product.
- Avoid first-person plural product voice. Name the product or user role instead.
- Use present tense and active voice.
- Use sentence-case headings.
- Keep paragraphs short.
- Avoid filler phrases, hedging, and marketing language.
- Avoid weak instruction forms. Use "must" for requirements, "use" for actions, and "can" only for options.
- Use "ensure" for verification language.
- Use "select" instead of "click" in UI guidance.
- Do not use em dash or en dash characters. Use commas, periods, colons, parentheses, or ASCII hyphens.
- Code examples must be complete, copyable, and safe to paste.
- Use placeholders such as `<API_KEY>`, `<TOKEN>`, `<PROJECT_ID>`, `<RUN_ID>`, and `<REDACTED>` for sensitive values.

## Concept and terminology rules

Use the same concept names in code, schemas, command help, docs, tests, and errors.

| Term          | Meaning                                                                            | Do not confuse with                                       |
| ------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Agent         | AI runtime primitive that accepts messages, tools, context, and emits AG-UI events | Workflow or task                                          |
| Tool          | Callable capability used by an agent or workflow step                              | MCP tool unless the surface is explicitly MCP             |
| Workflow      | Step graph or DAG for multi-step execution                                         | Task or schedule                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veryfront/veryfront-code](https://github.com/veryfront/veryfront-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
