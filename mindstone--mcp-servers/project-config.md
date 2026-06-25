---
trigger: always_on
description: A monorepo of independent, source-available MCP (Model Context Protocol) servers. Each subdirectory of `connectors/` is its own npm package with its own tests, CHANGELOG, and release cadence.
---

# AGENTS.md

A monorepo of independent, source-available MCP (Model Context Protocol) servers. Each subdirectory of `connectors/` is its own npm package with its own tests, CHANGELOG, and release cadence.

**This is a public, source-available repo.** No secrets, internal URLs, ticket references, or real customer/company names belong in source, docs, or commit messages — use fictional placeholders (`Acme Corp`, `jane@example.com`) in examples and fixtures.

See `README.md` for setup, `CONTRIBUTING.md` for the contribution workflow, and `SECURITY.md` for vulnerability reporting.

## Operating principles

- Smaller diffs over sweeping rewrites. Modular over monolithic.
- Match existing patterns before inventing new ones. Reuse before duplicating.
- Treat safety and reversibility as non-negotiable; treat speed as negotiable.
- Plan the smallest coherent change, challenge it for hidden risk, run it, verify with the project's own tests, then report.
- Before acting, check for a closer `AGENTS.md` inside the directory being edited — those rules override this file within their scope.
- When reporting completion, surface remaining follow-ups, unresolved questions, and out-of-scope observations the human should know about.

## Repository layout

```
connectors/
  _template/         Starter template. Copy this when adding a new connector.
  <connector-name>/  One independent npm package per directory.
test-harness/        Shared test utilities, linked via `file:` dependency.
scripts/             Repo-wide maintenance scripts. Local-only by design.
docs/                Internal docs (security audits, branch protection, etc.).
.github/             CI workflows and issue/PR templates.
```

There is no root-level workspace orchestration. Always `cd` into the specific connector being worked on.

## Scope discipline

A change touches exactly one connector unless the task is explicitly repo-wide (CI workflows, `test-harness/`, or root policy files). If drift is spotted in a sibling connector, flag it in the PR description; do not fix it inline. Do not modify `connectors/_template/` unless the task is to update the template — changes there propagate to every future connector.

## Code conventions

- TypeScript, strict mode. Do not silence the compiler with `any`; fix the type instead.
- Validate every tool input and external response with Zod. No hand-rolled runtime validation.
- Build servers and tools with `@modelcontextprotocol/sdk`. Do not invent a parallel protocol layer.
- Tests use Vitest and the shared `test-harness/`. Every new tool ships with smoke, happy-path, and error tests.
- Keep runtime dependencies minimal. New deps require justification in the PR description.
- Add a comment only when the reason behind code is non-obvious. Do not narrate what well-named code already says.
- No emojis in source, commits, or PR bodies unless a user-facing tool description explicitly requires one.

## Security invariants

These are load-bearing. Do not relax, bypass, or refactor any of them without an explicit human-approved security review.

1. The repo-root `.npmrc` enforces `min-release-age=7`. Do not lower the value, comment it out, or override it per-workflow.
2. Every GitHub Action in `.github/workflows/` is pinned to a commit SHA, kept current by Dependabot. Do not pin to a tag (`@v4`) or a branch (`@main`).
3. Every workflow job declares a least-privilege `permissions:` block. Do not use `permissions: write-all` or rely on defaults.
4. Local OAuth callback servers bind to `127.0.0.1` only. Do not honour any env var that could rebind to `0.0.0.0` or an external interface.
5. File-reading and file-uploading connectors constrain reads to `MCP_WORKSPACE_PATH` (or `os.tmpdir()`) using canonical-prefix containment that handles symlinked roots. Do not replace with substring checks or non-canonical `startsWith`. The same canonical-prefix discipline applies to download targets and any path joined from external input.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindstone/mcp-servers](https://github.com/mindstone/mcp-servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
