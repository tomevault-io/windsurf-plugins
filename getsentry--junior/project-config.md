---
trigger: always_on
description: Use **pnpm**: `pnpm install`, `pnpm dev`, `pnpm test`, `pnpm typecheck`, `pnpm skills:check`
---

# Agent Instructions

## Package Manager

Use **pnpm**: `pnpm install`, `pnpm dev`, `pnpm test`, `pnpm typecheck`, `pnpm skills:check`

## Commit Attribution

AI commits MUST include:

```
Co-Authored-By: (agent model name) <email>
```

## File-Scoped Commands

| Task                  | Command                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------- |
| Unit test file        | `pnpm --filter @sentry/junior exec vitest run path/to/file.test.ts`                                 |
| Integration test file | `pnpm --filter @sentry/junior exec vitest run path/to/file.test.ts`                                 |
| Eval file             | `pnpm --filter @sentry/junior-evals exec vitest run -c vitest.evals.config.ts path/to/eval.test.ts` |

## Key Conventions

- Use `/commit` skill for any commit operation.
- Use `/create-pr` skill for any PR creation operation.
- Use `/skill-creator` skill when creating or updating skills.
- Prefer integration tests for most product/runtime changes that need real wiring.
- Use evals as the integration-style layer for agent/prompt/natural-language behavior. See `packages/junior-evals/README.md`.
- Run evals from Codex as escalated host commands when they need real Vercel Sandbox/network access; use `pnpm evals` for the full suite.
- Use instrumentation conventions from `specs/logging/index.md`.
- Use OpenTelemetry semantic keys for logs; when no semantic key exists, use `app.*`.
- Keep release package lists aligned across `.craft.yml`, `scripts/bump-release-versions.mjs`, `.github/workflows/ci.yml`, `README.md`, and release docs; verify with `pnpm release:check`.
- Minimize defensive programming — no fallbacks when systems are expected to work. Ensure errors are captured correctly. Use retries for expected network failures, nothing more.
- Prefer minimal interfaces and simple components across the codebase.
- Keep public surfaces small: fewer exported types/functions, fewer integration points, explicit contracts.
- Prefer composition over abstractions that add indirection without clear reuse.
- Prefer standards/library-native patterns before custom infrastructure.
- Prefer standards-based streaming surfaces over custom transport loops.
- Chat SDK streaming standard: pass `AsyncIterable<string>` to `thread.post(...)`.
- Pi SDK streaming standard: consume `Agent` events (`message_update`/`text_delta`) and bridge deltas into the `AsyncIterable` shim.
- Avoid bespoke Slack `chat.update` loops unless required by a hard platform limitation.
- Prefer hard cutover for command or skill renames and behavior migrations unless backward compatibility is explicitly requested.
- Prefer integration tests over unit tests when real runtime wiring is needed and the contract is not model interpretation itself.
- Use evals when the contract is agent-facing behavior, prompt interpretation, natural-language routing, continuity, or reply quality.
- Use unit tests only for small local deterministic logic and algorithms.
- If a test needs multiple mocks to prove a user-visible workflow, it is probably in the wrong layer.
- Logs, spans, status telemetry, and other monitoring output are not behavior contracts. Do not add assertions for them, and do not mock logging/monitoring modules unless the test is explicitly about instrumentation.

## Engineering Principles

- Optimize for obvious code over flexible-but-indirect abstractions.
- Keep public interfaces small and intention-revealing.
- Let file/module structure carry context so names do not have to.
- Keep exported names role-specific; keep local helper names short.
- Prefer domain language over mechanism language.
- Every exported function must have a brief JSDoc comment explaining its intent (the _why_, not the _what_).

## Policies

- `policies/README.md` (when to add a policy doc and how policy docs should stay scoped)
- `policies/code-comments.md` (repo default for code comments, docstrings, and exported-function JSDoc)
- `policies/policy-template.md` (template for adding new policy docs)

## Investigation-First Development

- Before implementing anything that depends on an external system (Vercel, Slack, bundler, framework), read the relevant documentation or source code first. State the constraint being relied on before writing code.
- Before removing an architectural layer, prove the replacement handles all known edge cases in a working proof-of-concept. Do not remove the incumbent until the replacement is verified end-to-end.
- When changing a function signature, error contract, or shared pattern, grep for all consumers and verify each one still works. Do not assume fixing one call site is sufficient.
- If a fix attempt fails, stop. Re-read the error, trace the full system from input to output, and identify the root cause before trying another fix. Do not commit progressive patches that address symptoms layer-by-layer.
- When implementing message handling or Slack interactions, explicitly verify both DM and channel paths, and both first-delivery and retry paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/junior](https://github.com/getsentry/junior) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
