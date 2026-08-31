---
trigger: always_on
description: Use this guide when you change this repository. User documentation lives in
---

# stim-cli agent guide

Use this guide when you change this repository. User documentation lives in
[`packages/stim-cli/README.md`](./packages/stim-cli/README.md).

## Project

stim-cli gives each React Native or Expo workspace an isolated Metro port and an
owned simulator or emulator. It caches native builds and captures structured
logs.

The normal flow is:

```text
worktree create -> start -> ios|android -> logs --errors -> stop -> worktree remove
```

The command surface is `doctor`, `worktree create|remove`, `start`, `stop`,
`ios`, `android`, `logs`, `status`, `gc`, and `guide`. Do not add commands or
flags without an explicit product decision. Projects can wrap stim-cli when they
need custom behavior.

Runtime state belongs under `$STIM_CLI_HOME/workspaces/`, not in the project
tree. Do not restore `init`, project setup mutations, or the deleted
`stim-cli-init` skill. `doctor` reports setup that requires project judgment.

## Development

The repository is a pnpm workspace. Published packages live under `packages/`.
The packages are ESM-only. They require Node.js 20.19.4 or later on Node 20,
or Node.js 22.12.0 or later. Repository development requires Node.js 22.18.0
or later because tsdown uses that floor.

```bash
pnpm install
pnpm run format:check
pnpm run lint
pnpm run build
pnpm run typecheck
pnpm test
```

Use only the checks that apply while iterating. Run all defined checks before a
commit. Run `pnpm run test:e2e` when a change affects an end-to-end workflow.

## Issue and pull request workflow

When you find a bug or improvement, search the open GitHub issues first. If no
issue already describes it, create one before implementation. Refresh the
remote refs, then confirm that an existing issue still applies to current
`origin/main`; close stale issues with the fixing commit and verification
evidence instead of creating duplicate work.

Implement each valid issue in its own git worktree and branch created from the
refreshed `origin/main`. Independent issues may run in parallel worktrees. Keep
the branch limited to that issue, run the required checks, and open a pull
request that links the issue.

As soon as the pull request is open, assign a fresh agent that did not implement
the change to review the issue, diff, tests, and user-facing guidance. Address
every actionable finding and rerun the affected checks. Mark the pull request
ready only after that fresh review is clear. Merge only after all required CI
checks pass; if CI fails, fix the branch, repeat the review when behavior
changes, and wait for the new checks.

## Architecture rules

- **Single exec wrapper.** Route all child processes through
  `packages/stim-cli/src/exec.ts`. Use `runFile` for user-controlled paths.
- **Pure parsing and decision logic.** Keep parsers and selectors separate from
  thin I/O wrappers. Unit-test the pure functions.
- **Locked state.** Lock every read-modify-write to global config or workspace
  state. Use atomic writes. Long-lived build locks use PID liveness, not mtime.
- **Cache contracts.** The cache packages must work without `stim-cli` installed.
  Keep their config path, cache root, cache key, and registration behavior
  aligned with the CLI. Resolution order is environment, machine config, then
  default. Update `cache-packages.test.ts` when those rules change.
- **Source format.** Keep files under `src/`, `bin/`, and `test/` ASCII-only.
  Markdown can use Unicode.
- **Concurrency limits.** Build and device caps are opt-in through config or
  environment variables. Do not add a config command.

## Comment policy

Treat every code comment as removable. Keep a comment only when it is one of
these exceptions:

- A legal or license header.
- A non-obvious constraint from an external dependency, platform, vendor, or
  protocol. Name the external source and the concrete constraint.
- A required formatter directive such as `prettier-ignore`.
- A doc comment that defines a public API contract.
- A direct issue or RFC link for a constraint that code cannot express.

Delete narration, banners, commented-out code, workaround essays, and comments
that restate the code. Words such as `IMPORTANT`, `do not remove`, and `fine for
now` do not make a comment valid. Read the nearby code before judging a comment.
When no keep rule clearly applies, delete the comment.

Treat suppressions such as `eslint-disable`, `@ts-ignore`, and
`@ts-expect-error` as code problems. Keep a suppression only for a faulty,
pedantic, or style-only rule. If the rule protects correctness or safety, remove
the suppression and report the exact symbol as `MUST KILL` for a later refactor.
For a surprising behavior in this codebase, delete the explanation and report
the exact symbol as `MUST KILL`. Do not add `MUST KILL` markers to source files.

## Required invariants

### 1. Keep agent guidance current

Treat `packages/stim-cli/skill/SKILL.md` as a compact entry point, not a manual.
Keep it under 1,200 words. Include only the normal local workflow, permanent
ownership and deletion rules, and routing to `guide` topics. Put exact flags,
payload schemas, uncommon backends, release builds, cache mechanics, settings,
capacity details, cleanup internals, and remedies in `guide`. Do not duplicate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appandflow/stim](https://github.com/appandflow/stim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
