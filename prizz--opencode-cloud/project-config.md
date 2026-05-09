---
trigger: always_on
description: Before creating any git commit, you MUST run `just pre-commit`.
---

# Claude Code Instructions

## Pre-Commit Requirements

Before creating any git commit, you MUST run `just pre-commit`.

Only proceed with the commit if it passes. If it fails, fix the issues first.

**Exception:** If the commit contains only documentation and markdown changes (`.md` files), you may skip `just pre-commit`.

`just pre-commit` intentionally runs generation and formatting steps that may modify files you did not edit directly (including markdown). Treat these diffs as expected outputs of the check pipeline.

`just pre-commit` also runs `just check-opencode-guardrails-autofix`, which may auto-sync fork-boundary manifest drift in `packages/opencode/docs/upstream-sync/fork-boundary-manifest.json` when drift is the only issue.

If those diffs are mechanical (generation/format/lint output only), they should be committed with the related change. Do not treat them as noise and do not leave them uncommitted.

Common expected examples (non-exhaustive):
- `packages/opencode/packages/sdk/openapi.json`
- `packages/opencode/README.md` (including normalized markdown formatting / generated sections)
- `packages/core/README.md` (synced by hook)
- `bun.lock` files

Required post-`just pre-commit` checklist:
1. Check `git status` (superproject) and `git -C packages/opencode status` (submodule).
2. Review all newly changed files and confirm they are mechanical (no unintended behavior changes).
3. Commit/push submodule generated changes first when present.
4. Run `just update-opencode-commit`.
5. Commit/push the superproject pointer + Dockerfile pin update when changed.

## Bun Lockfile Updates

`bun.lock` file updates are expected in this repository across all `bun.lock` files when changing versions of our own packages (for example `packages/cli-node`, `packages/core`, and related workspace packages). They can also be produced by normal `just pre-commit`/build flows.

When these `bun.lock` changes are tied to intended package/version updates, they are valid and should be committed with the related change. Do not treat them as unexpected noise.
This same expected-and-commit rule applies to other mechanical formatter/generator outputs from `just pre-commit`, including markdown rewrites.

## Project Structure

This is a polyglot monorepo with Rust and TypeScript:

- `packages/core/` - Rust core library with NAPI-RS bindings
- `packages/cli-rust/` - Rust CLI binary
- `packages/cli-node/` - Node.js CLI wrapper
- `packages/opencode/` - Git submodule pointing to our fork of the opencode repository. We own this fork and can freely modify, commit, and push changes to it.

## Fork Isolation (Upstream Modification Policy)

The `packages/opencode/` submodule is a fork that syncs from upstream every 2 hours. To minimize merge conflicts:

- **Prefer putting new code in `fork-*` packages** (`fork-auth`, `fork-config`, `fork-ui`, `fork-security`, `fork-terminal`, `fork-cli`, `fork-provider`, `fork-tests`). These are ours and never conflict with upstream.
- **Minimize modifications to non-fork packages** (`app`, `opencode`, `sdk`, `ui`, `util`). When changes to upstream code are needed, keep the diff as small as possible — typically just an import and a single function call that delegates to a `fork-*` package.
- **Do not refactor upstream code.** Even if upstream code has bugs or poor patterns, fix only what directly blocks our users. Leave broader cleanups to upstream — they will fix their own bugs over time. Every line we change in upstream files is a potential merge conflict.
- **Tolerate imperfect upstream code.** If an upstream bug doesn't affect our users, leave it alone. If it does affect our users, fix it with the smallest possible diff (inline changes > new imports > new files).
- Use thin re-export shims in upstream packages that delegate to `fork-*` implementations when hooks are needed.

## Key Commands

```bash
just setup       # First command after clone/worktree init (hooks + deps + submodule bootstrap)
just build       # Build all packages
just test        # Run all tests
just e2e         # Run e2e tests (Playwright, boots server in-process)
just fmt         # Format all code
just lint        # Lint all code
just pre-commit  # Format, lint, build, test, and e2e
just clean       # Clean build artifacts
just run <args>  # Run CLI with arguments (e.g., just run status)
just dev         # Start local runtime (local submodule + cached sandbox rebuild)
```

Run `bash scripts/check-dev-prereqs.sh` to verify all required and optional tools are installed. Then run `just setup` before any other project command after cloning this repo. In new worktrees, initialize the submodule first with `git submodule update --init --recursive`, then run `just setup`.

Setup reference:
- Rust toolchain: `1.89` (from `rust-toolchain.toml`)
- Bun: `1.3.9+`
- Optional tools (`docker`, `jq`, `shellcheck`, `actionlint`, `cfn-lint`) are required only for specific flows (`just dev`, `just lint`, and CloudFormation hook checks).
- Rerun `just setup` for new clones/worktrees, if hooks are reset, or if dependency bootstrap looks stale.

## Git Hooks

This repo uses git hooks (wired via `git config core.hooksPath .githooks`, set up by `just setup`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pRizz/opencode-cloud](https://github.com/pRizz/opencode-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
