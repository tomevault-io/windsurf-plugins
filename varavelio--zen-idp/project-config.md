---
trigger: always_on
description: Zen IdP is a declarative, zero-maintenance OIDC Identity Provider.
---

# Agent Context for Zen IdP

## Summary

Zen IdP is a declarative, zero-maintenance OIDC Identity Provider.

## How to Maintain This Document

Keep this file current and minimal. Update it only when repository-wide workflow, structure, or agent guidance changes. Do not turn it into a changelog. Use it exclusively to indicate truly relevant things in the codebase; don't include any minor details that are obvious or don't warrant documentation.

## Required Agent Behavior

- Always check available Skills and MCPs before acting so you know what capabilities are available for you to use.
- Always read `Taskfile.yml` to understand available `task` commands for the project; do not list those commands here.
- When assigned a task, do not respond or stop until the requested task is complete.
- Run `task ci` for code checks. If it fails, fix failures caused by your own changes until it passes; stay within the scope of your changes and ignore pre-existing unrelated failures.
- All code, code comments, inline documentation, commit messages, and any other text in the project MUST be written in English.

## Testing

Whenever possible, write tests that verify the expected behavior of the code being implemented. You must follow the following rules regarding testing:

- Write the unit tests close to the code they are testing; for example, if you have the file foo.go, you have to put all the unit tests inside foo_test.go.
- When creating tests for Go, use the testify package which is already installed in the project. Prioritize using "require" whenever possible instead of "assert" so that the tests fail quickly when something is wrong.
- Write high-value tests, focus on critical logic and relevant edge cases. Quality beats quantity; don't write tests just to inflate coverage; make sure every test adds real value.
- Treat tests as our primary tool to catch regressions. Write every test to guarantee long-term stability, correctness, functionality, and maintainability as the codebase evolves.
- Group test cases with subtests (Go): Keep all test cases for a given function inside a single top-level Test function using subtests (t.Run). This maintains a clean structure and avoids file clutter when testing multiple functions in the same file.

## End-to-end testing

The `e2e/` directory holds two end-to-end suites, both orchestrated by
`task e2e` from the Taskfile.yml (also part of `task ci`):

- `e2e/http`: the black-box HTTP suite in Go, compiled only under the `e2e`
  build tag. Each complete product scenario lives in its own `*_test.go`
  file and drives the compiled binary only through its CLI and HTTP surface.
- `e2e/browser`: the browser suite in Deno using Playwright. Each complete
  product scenario lives in its own `*.test.ts` file and drives a real
  browser against the compiled binary.

Both suites keep their reusable plumbing in a `harness/` subdirectory
(`e2e/http/harness` and `e2e/browser/harness`), so shared code is never
repeated inside individual tests. The HTTP harness spawns one isolated
instance per test (configuration, state database, and loopback port),
reproduces the crypto contracts independently, and never imports
`internal/` packages, so the suite always validates public behavior as a
black box.

## Data Storage

- Generated record identifiers used as SQLite primary keys MUST be TypeID values stored as TEXT.
- Operator-declared natural keys, such as the YAML `sub` identifier, remain as declared and are not converted to TypeIDs.

## Documentation site

The project's docs page is built with Veta, a static site generator, using the veta-theme-vara theme. The site configuration lives in `./docs` (entry point `veta.yaml`) and the actual content in `./docs/content`. Read all the `.js` files inside `docs/pages` to understand how the pages are generated.

Read these references if you have doubts about how to use them:

- Veta: https://raw.githubusercontent.com/varavelio/veta/main/README.md and https://veta.varavel.com/docs/llms.txt
- veta-theme-vara: https://raw.githubusercontent.com/varavelio/veta-theme-vara/main/README.md and https://vara.varavel.com/docs/llms.txt

## Project structure

This is a Go project with a structure that aims to remain flat, simple, and idiomatic; here's a summary:

- cmd/zen-idp: The entry point of the program
- scripts/release: Cross-compiles the release archives for every supported platform and writes their manifest and checksums into dist
- installers: The published installation integrations: the Homebrew formula generator and the shell and PowerShell installers that the release workflow feeds with the release binaries
- internal/admin: Authenticates the administrator, creates the distinct administrator sessions that gate the administrative interfaces, and records authentication and rate-limit events
- internal/audit: Records security-relevant operational events as disposable SQLite-backed audit records and enforces their retention by purging records older than the configured deadline
- internal/cli: Parses Zen IdP command-line invocations into typed commands
- internal/clock: Formats and parses the canonical UTC RFC 3339 timestamps used by the state database
- internal/clockcheck: Rejects implausible system clock conditions at startup so the service fails safely

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varavelio/zen-idp](https://github.com/varavelio/zen-idp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
