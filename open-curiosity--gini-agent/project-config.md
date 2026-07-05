---
trigger: always_on
description: These instructions apply to the whole repository unless a nested `AGENTS.md` overrides them for a subtree.
---

# Gini Agent Instructions

These instructions apply to the whole repository unless a nested `AGENTS.md` overrides them for a subtree.

## Shape

Gini is a Bun TypeScript personal agent runtime. The gateway owns durable state and execution; CLI, Next.js, future mobile, MCP, messaging, and scripts are clients of the same `/api/*` contract.

The repository is a Bun workspaces monorepo: the root `package.json` is a private workspace root (single `bun.lock`, shared dependency versions via the workspace `catalog`), with `packages/runtime` (`@gini/runtime`, the gateway + CLI), `packages/web` (`@gini/web`, the Next.js control plane), and `packages/mobile` (`@gini/mobile`, the Expo app). Bundled `skills/`, `docs/`, `scripts/`, `vendor/`, and `patches/` live at the repository root, which the runtime discovers by walking up to the workspace marker (`projectRoot()` in `packages/runtime/src/paths.ts`). Run `bun install` once at the root — it covers every package. See ADR bun-workspaces-monorepo.md.

Start with `README.md` for the docs index. Keep `docs/whitepaper.md`, `docs/architecture-overview.md`, focused docs, and `docs/adr/` in sync with architecture changes.

## ADRs

Keep ADRs current when architecture changes.

- Update an existing ADR when the original decision still stands but implementation details, consequences, or acceptance checks changed.
- Add a new ADR for a significant architecture decision, trust boundary, persistence model, process shape, provider strategy, client contract, or operational workflow.
- If a change makes an ADR obsolete, mark the old decision as superseded and link to the replacement ADR.
- ADRs should be forward-looking. Don't write removal logs — context for what was deleted belongs in git history and PR descriptions, not in an ADR. If two ADRs overlap because of a consolidation, merge the canonical forward-looking content into one and delete the redundant ADR (instead of leaving both).
- ADRs are named by slug (`docs/adr/<slug>.md`), no number prefix. Pick the slug carefully and never rename it once merged — the filename is the citation key.
- Always cite an ADR by its full filename including `.md` so the reference is unambiguously a file: `see ADR agent-memory-isolation.md` in prose and code comments, and `[Per-Agent Memory Isolation](docs/adr/agent-memory-isolation.md)` for markdown links.

## Boundaries

- Prefer existing module patterns over new abstractions.
- API handlers should delegate behavior to bounded runtime modules (`packages/runtime/src/execution`, `packages/runtime/src/memory`, `packages/runtime/src/jobs`, `packages/runtime/src/hooks`, `packages/runtime/src/governance`, `packages/runtime/src/capabilities`, `packages/runtime/src/integrations`, `packages/runtime/src/runtime`).
- Storage and low-level persistence belong in `packages/runtime/src/state/*`.
- CLI commands should prefer the public runtime API for product behavior.
- Browser code must not receive gateway bearer tokens; token injection stays server-side in the Next.js BFF.
- Side-effecting tools must preserve approval, audit, and trace behavior.
- Instance-aware paths, ports, logs, and state must remain isolated.
- Skill scripts (`skills/**/scripts/*.ts`) are first-class code: typechecked via the root `tsconfig` and run by `bun run test` (which spans `./skills`). Put a script's tests in `<skill>/scripts/__tests__/` — never directly in `scripts/`, which the loader advertises as runnable scripts — and keep scripts self-contained (no `packages/runtime/src/` imports) so the skill stays portable; export a pure function and import it from the test when you need coverage.

## Branches

Use `<type>/<kebab-case-topic>`, where `<type>` is one of `feat`, `fix`, `chore`, `docs`, `refactor`, or `test`. Examples: `feat/profile-switcher`, `fix/chat-title-overflow`, `docs/release-process`.

## Commits and PR titles

Commit messages and PR titles describe the technical change, not the process that produced it. Public history is what reviewers and future readers see; the back-and-forth that shaped the diff is internal.

Don't write:

- `Address codex review round 3: ...`
- `Round-2 fix: ...`
- `Apply review feedback`
- `Fix bugs from <reviewer name>`
- `Sanitize PR-review meta-narration` — even meta-cleanup messages can leak the meta

Do write:

- `Sanitize extraBody and tighten provider parsers`
- `Per-provider baseUrl defaults; tighten CLI flag accuracy`
- `Tighten provider and provider-test comments`

The same rule applies to **comments inside source and tests**: describe the hazard generically (what the test pins, why a guard exists) rather than its history (`Round-3 caught that…`). Grep your diff for `[Rr]ound[- ]?[0-9]` and `review` in comments before pushing.

If iterating with multiple review-fix commits before the PR lands, squash to a clean narrative first (`git rebase -i`, or use squash-merge). Once the PR is merged, the messages are permanent.

## Verification

For code changes, run relevant tests plus broader checks when practical:

```bash
bun run typecheck
bun run test
bun run gini smoke
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Curiosity/gini-agent](https://github.com/Open-Curiosity/gini-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
