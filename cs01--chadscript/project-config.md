---
trigger: always_on
description: **ALWAYS work on a git worktree and branch. NEVER modify files directly on `main`.** `main` must always remain clean. Every piece of work — features, bug fixes, docs, even CLAUDE.md edits — must happen on a dedicated branch in a worktree:
---

# ChadScript Rules

## Worktree Rule

**ALWAYS work on a git worktree and branch. NEVER modify files directly on `main`.** `main` must always remain clean. Every piece of work — features, bug fixes, docs, even CLAUDE.md edits — must happen on a dedicated branch in a worktree:

```bash
git worktree add .worktrees/<name> -b <branch-name>
cd .worktrees/<name>
# do work, commit, then open a PR
```

## Autonomous PR Workflow

Agents can work autonomously end-to-end: create worktrees, make changes, push branches, create PRs,
monitor CI, and merge when green. You have push access to feature branches and merge access to PRs.

1. Create a worktree and branch
2. Make changes, run `npm run verify:quick`, commit
3. `git push origin <branch>` — push to remote
4. `gh pr create` — open a PR
5. `gh pr checks <number>` — monitor CI
6. When CI is green: `gh pr merge <number> --squash --delete-branch` — merge to main
7. Clean up: `cd /Users/csmith/git/ChadScript && git worktree remove .worktrees/<name>`
8. Pull main and continue with next task

**Every PR must be seen through to completion** — don't just open and walk away. Monitor CI, fix failures,
merge when green, delete the remote branch, and remove the local worktree.

**Never push to main directly.** Always go through PRs.

## PR title + body format

**Title**: `[topic] change` — topic is the area touched (e.g. `codegen`, `parser-native`, `ci`, `stdlib/net`, `llvm-builder`). Change is the one-line summary of what this PR does. **Never** use internal step/phase names like "phase 3 step 2" or "step4-phase1b" in the title — those mean nothing to anyone reading the PR list.

Good:
- `[codegen] seed symbol table on typed let-decl without init`
- `[parser-native] restore trailing return in bare switch case bodies`
- `[llvm-builder] add 5 remaining opcode wrappers`

Bad:
- `step4-phase1b: remaining ops`
- `phase-e step 5 part 3 follow-up`

**Body** must include **Before** and **After** sections describing the user-facing change (or, if purely internal, the behavior difference before vs after). Add `## Description` with any additional context. Example:

```
## Before
Compiler failed with "Method 'on' on 'sock' is not supported" when using `let sock: Socket;` followed by a conditional assignment.

## After
`sock.on(...)` resolves correctly via the declared `Socket` interface regardless of how `sock` is later assigned.

## Description
Seeds the symbol table with the declared type at decl-without-init sites in variable-allocator. ~6 LOC fix.
```

If the PR is purely internal (no user-visible effect), state that explicitly in **After**: `No user-facing change. Internal refactor only.`

## Testing & Commit Workflow

After completing each todo:

1. Run unit tests
2. If tests pass, commit the changes
3. If tests fail, fix them before moving to the next todo
4. Never move on to the next todo while tests are failing

## Self-Hosting Verification

Before considering any feature complete, run the full self-hosting chain:

1. `npm run verify` — runs tests and self-hosting in parallel (preferred)
2. `npm run verify:quick` — same but skips Stage 2 (day-to-day dev)

Or manually:

1. `npm test` — all tests pass (auto-uses native compiler if `.build/chad` exists)
2. `bash scripts/self-hosting.sh` — full 3-stage self-hosting
3. `bash scripts/self-hosting.sh --quick` — skip Stage 2

New features have complex side effects that may not be caught by unit tests alone. A change that passes all tests can still break self-hosting. The Stage 2 test is the true verification — it proves the compiler's output is correct enough to compile itself.

## Versioning & Releases

Version is defined in one place: `package.json`. `npm run build` auto-generates `src/version.ts` via the `prebuild` script (`scripts/gen-version.js`). Both `chad-node.ts` and `chad-native.ts` import `VERSION` from there.

To bump a version:
1. Edit `version` in `package.json`
2. `npm run build` (regenerates `src/version.ts`)
3. Merge to main
4. `git tag v<version> && git push origin v<version>` — CI creates a GitHub Release with binaries

## Stale Native Compiler

After a rebase or merge that brings in new codegen features, `.build/chad` becomes stale — it was compiled
from the old source and doesn't know how to compile the new features. **Rebuild it**:

```bash
rm -f .build/chad && node dist/chad-node.js build src/chad-native.ts -o .build/chad
```

Tests auto-detect `.build/chad` and use it over `node dist/chad-node.js`. A stale native compiler causes
mysterious test failures that pass fine with the node compiler.

## Worktree Setup

Each worktree builds its own `vendor/` — do not symlink it from another worktree or the main repo. Different branches may have different `c_bridges/` sources, and a shared vendor dir causes races and silent corruption when multiple agents build concurrently.

```bash
bash scripts/build-vendor.sh
```

`npm test` rebuilds `dist/` only if `src/` is newer, and builds `.build/chad` only if missing.

# ChadScript Architecture Guide

## What It Is


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cs01/ChadScript](https://github.com/cs01/ChadScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
