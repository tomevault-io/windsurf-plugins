---
trigger: always_on
description: - The shipping `climon` **client** is the **Rust** workspace under `rust/` (crates `climon-cli`, `climon-session`, `climon-pty`, `climon-store`, `climon-config`, `climon-logging`, `climon-proto`, `climon-remote`, `climon-install`, `climon-update`). **All client work — new features and bug fixes — goes in `rust/`.**
---

# Copilot instructions for climon

## Client = Rust, server = Bun (read this first)

- The shipping `climon` **client** is the **Rust** workspace under `rust/` (crates `climon-cli`, `climon-session`, `climon-pty`, `climon-store`, `climon-config`, `climon-logging`, `climon-proto`, `climon-remote`, `climon-install`, `climon-update`). **All client work — new features and bug fixes — goes in `rust/`.**
- The Bun/TypeScript client that used to live under `src/` (`src/index.ts`, `src/launcher.ts`, `src/session-host.ts`, `src/pty.ts`, `src/cli/`, `src/client/`, `src/daemon/`, `src/install/`, most of `src/remote/`, …) has been **removed** — it was rewritten in Rust. Do **not** try to run or restore it; all client work goes in the `rust/` crates. The TypeScript that remains under `src/` is the maintained dashboard server/web plus shared support modules (configuration, logging, i18n, session defaults, the remote-ingest helpers the server imports, and `src/update/pubkey.ts`, the Ed25519 key read by `rust/climon-update` at build time).
- The dashboard **server** (`climon-server`, built from `src/server.ts` with `src/server/` and `src/web/`) is **NOT legacy**. It is still Bun, still maintained, and is never rewritten. The Rust client interoperates with it byte-for-byte over the shared metadata/socket/config surfaces.
- See `docs/architecture.md` for the component breakdown.

## Workflow

- Always start new work in a fresh git worktree under the `.worktrees/` folder, never directly on the main or dev checkout. Create one per task with `git worktree add .worktrees/<branch-name> -b <branch-name>` (or check out an existing branch) and do all edits, builds, and tests there. The `.worktrees/` folder is gitignored.
- **Always open pull requests against the `dev` branch, never `main`.** `dev` is the integration branch; feature/fix PRs are squash-merged into it. Releases are **tag-driven**: they ship only when a `vX.Y.Z` tag is pushed, so a `dev` → `main` merge alone no longer cuts a release. See `docs/cutting-a-release.md` for the full runbook.
- **Always squash merge into `dev` unless explicitly told otherwise.** When merging a PR into `dev` (e.g. `gh pr merge --squash`), use a squash merge so each feature/fix lands as a single commit and keeps `dev` history linear. Only use a different merge strategy when the user specifically requests it.
- **Never squash merge `dev` into `main`, and never squash the automated `main` → `dev` back-merge PR.** Release/hotfix → `main` merges and the back-merge PR the Release workflow opens must all use a real merge commit (`gh pr merge --merge`), never `--squash` or `--rebase`, so `main` and `dev` keep a shared ancestor and the next release diff stays clean.
- **When cutting a release, first read [`docs/cutting-a-release.md`](../docs/cutting-a-release.md) and follow it exactly.** It is the authoritative runbook for the tag-driven flow. In short: prepare the release on a `release/*` (or `hotfix/*`) branch, bump the version there with `bun run release` (or bump `package.json` and the CLI fixtures in lockstep), merge to `main` with a real merge commit, then push the `vX.Y.Z` tag — pushing the tag is what ships. The golden CLI fixtures `fixtures/cli/version.txt` and `fixtures/cli/help.txt` embed `climon v<semver>` on line 1 and are asserted byte-for-byte by `rust/climon-cli/tests/cli_fixtures.rs`; the Release workflow's `version` job also refuses to ship unless the tag matches them. `scripts/release.ts` rewrites all three together; if you edit `package.json` by hand you MUST also bump the version token on line 1 of both fixtures, or `cargo test` (and CI) goes red.

## Local Copilot CLI skills

- Repo-local Copilot CLI skills live in `copilot-plugin/` (a per-session plugin, not installed globally). Load them with `copilot --plugin-dir copilot-plugin` from the repo root, then invoke a skill by describing the task (e.g. "update the changelog" runs the `update-changelog` skill). See `copilot-plugin/README.md`.

## Build, test, and lint commands

### Rust client (canonical — do client work here)

- The client crates live in `rust/`. Build with `cargo build` (or `cargo build --release`) from `rust/`.
- Run the Rust tests with `cargo test`; lint with `cargo clippy --all-targets`; format with `cargo fmt`.
- License/attribution gates use `cargo deny` and `cargo about` (`rust/deny.toml`, `rust/about.toml`).
- The shipped `climon` binary is built from `rust/climon-cli` and packaged by `scripts/compile.ts`.

### Bun server + tests

- Install dependencies with `bun install`. The project uses Bun (`packageManager: bun@1.3.10`) and TypeScript ESM.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jackgeek/climon](https://github.com/jackgeek/climon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
