---
trigger: always_on
description: Operating manual for AI coding agents working in this repository.
---

# AGENTS.md

Operating manual for AI coding agents working in this repository.

## About The Project

`cloakbrowser-mcp` is a stdio MCP bridge for upstream `@playwright/mcp`. It starts upstream Playwright MCP as a child process, injects the CloakBrowser Chromium executable through a generated Playwright MCP config, forwards upstream tools unchanged, and adds only two local introspection tools.

- Runtime: Node.js `^22.13.0 || >=24.0.0`, ES modules, TypeScript `strict` with `NodeNext`.
- Public surface: CLI package only, `bin: cloakbrowser-mcp`.
- Docker base: pinned official Playwright MCP image from `Dockerfile`.

## Golden Rules

1. Write the simplest possible code. Do only what was requested.
2. Use the `context7` MCP tool whenever you need up-to-date documentation or API references for external libraries.
3. Everything in this repository is written in English.
4. AI-agent reasoning and code-agent reasoning for this repository must be in English.
5. Do not copy, rewrite, or mutate upstream Playwright MCP browser tool contracts.

## Project Layout

```text
src/
  cli.ts                  CLI entry point
  server.ts               outer MCP proxy server
  index.ts                metadata export only
  bridge/                 config generation, env parsing, upstream path resolution, local tools
  cli/                    option handling, diagnostics, singleton cleanup
  http/                   Streamable HTTP server and session lifecycle
  logging/                stderr/file logging
  protocol/               shared protocol constants
  runtime/                console fallback source strings
  project/                project metadata
tests/
  unit/                   env/config/local tool tests
  integration/            fake-upstream MCP proxy tests
  fixtures/               fake upstream MCP server
```

## Daily Commands

```bash
npm run dev
npm test
npm run test:unit
npm run test:integration
npm run typecheck
npm run lint
npm run format
npm run format:check
npm run build
npm run package:verify
npm run docker:build
npm run docker:smoke
npm run bridge:compare
npm run check
```

`npm run check` must pass before any change is considered done.

## TypeScript

- Keep `strict` mode on.
- ESM only. Internal imports end with `.js`.
- Use configured aliases for internal imports instead of relative paths:
  `#src/...` for runtime source, `@/...` for source imports in tests,
  `@tests/...` for test helpers, and `#scripts/...` for scripts.
- Prefer explicit types and small pure functions.
- Do not use `console.*` in runtime code. CLI help/version may write to `process.stdout`; errors may write to `process.stderr`.
- Do not add `any`, `// @ts-ignore`, or non-null assertions to silence the checker.

## Bridge Rules

- Upstream Playwright MCP tools are forwarded unchanged.
- Local tools are limited to `cloakbrowser_binary_info` and `cloakbrowser_bridge_info`.
- `PLAYWRIGHT_MCP_*` is the primary configuration namespace.
- `CLOAK_PLAYWRIGHT_MCP_*` is only for bridge-specific Cloak toggles.
- Do not add `CLOAKBROWSER_MCP_*` aliases.
- Do not restore the old native adapter, custom capability model, origin policy, artifact manager, verify helpers, or custom browser tools.

## Tests

- Vitest.
- Unit tests live under `tests/unit/`.
- Integration tests live under `tests/integration/`.
- Use the fake upstream MCP server for proxy behavior.
- Tests must write only to `tmpdir()` paths they create and clean up.
- Prefer property-based tests for parsers, option normalization, environment
  handling, and other boundary-heavy pure logic.

## Agent Skills And Workflow Routing

Repository-owned skills live under `.agents/skills/`. Use the matching skill
when its frontmatter description routes the current request; do not invoke a
specialized workflow merely because its files exist.

- Review and refactoring: `code-review-and-quality`, `code-simplification`,
  `performance-optimization`, and `security-and-hardening`.
- Discovery and durable context: `context-engineering`,
  `documentation-and-adrs`, `project-docs-maintainer`,
  `doubt-driven-development`, `idea-refine`, and `interview-me`.
- Specification delivery: `spec-driven-development`,
  `planning-and-task-breakdown`, and `incremental-implementation`.
- GitHub delivery: `project-pull-request` and `project-release`.

The authoritative slash-command routes are:

- `/spec` -> `.agents/skills/spec-driven-development/SKILL.md`
- `/plan` -> `.agents/skills/planning-and-task-breakdown/SKILL.md`

Do not create a second implementation of either route. For substantial
workstreams, store the contract and execution artifacts under
`docs/specs/<slug>/` and follow:

- `.agents/references/specification-interview.md` for Prompt MCP interviews,
  decision persistence, recovery, and specification approval;
- `.agents/references/task-packets.md` for planning and one-packet execution.

Whenever a repository-owned skill needs a material user decision, use the
globally configured Prompt MCP instead of a plain-chat multiple-choice
question. Inspect the callable schema, use the repository's absolute path as
`workspace_path`, prefer `workspace` persistence for recoverable workflows,
and use stable semantic IDs. A cancellation, timeout, unavailability,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swimmwatch/cloakbrowser-mcp](https://github.com/swimmwatch/cloakbrowser-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
