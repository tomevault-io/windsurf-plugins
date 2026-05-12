---
trigger: always_on
description: All Nix builds, Firecracker operations, `mvmctl` runtime commands (anything that boots, talks to, or manages microVMs), and Linux-specific syscalls MUST be run inside the Lima VM. Use `limactl shell mvm-builder -- <command>` to execute commands inside the VM. The Lima VM name is `mvm-builder` (renamed from `mvm` in W7.2).
---

# Agent Working Agreement

## Lima VM Requirement

All Nix builds, Firecracker operations, `mvmctl` runtime commands (anything that boots, talks to, or manages microVMs), and Linux-specific syscalls MUST be run inside the Lima VM. Use `limactl shell mvm-builder -- <command>` to execute commands inside the VM. The Lima VM name is `mvm-builder` (renamed from `mvm` in W7.2).

**Run cargo on the macOS host wherever it compiles cleanly.** `cargo test`, `cargo check`, and `cargo build` should default to the host so worktrees don't deadlock on the single shared Lima VM (cargo target-dir contention, registry locks, and `.git/index` cross-mount races are real and have caused us to lose work). Tests that genuinely need Linux — vsock, jailer/seccomp, dm-verity, network namespaces, anything that pokes at `/dev/kvm` or `/proc/net` — should be gated with `#[cfg(target_os = "linux")]` and only those sub-targets are run inside Lima. Workspace-wide `cargo clippy --workspace --all-targets -- -D warnings` is still expected to pass inside Lima before merge, since clippy needs to see the Linux-gated code paths.

**git only runs from the main `mvm/` checkout, never from inside a worktree directory and never from inside Lima.** The main checkout is the single git operator for the whole repo. To act on a worktree's branch, use `git -C /path/to/.worktrees/mvm-<slug> <cmd>` from the main checkout — that drives the worktree's index/HEAD/refs while keeping the running git process anchored at the main checkout. Reasons: (1) only one git process at a time touches `.git/objects`, `.git/packed-refs`, and the shared `.git/hooks/` invocation context, eliminating the cross-worktree contention that has caused us to lose work; (2) the 9p/virtiofs share with Lima does not share git's lock semantics, so git from inside Lima deadlocks against host-side git. Cargo/nix/firecracker/mvmctl commands still run from each worktree's own directory — only `git` is centralized.

If the Lima VM is not running, boot it with:

```bash
cargo run -- dev
```

Once running, access it with:

```bash
limactl shell mvm-builder
```

Examples:
- `limactl shell mvm-builder -- cargo run --quiet -- build openclaw --force`
- `limactl shell mvm-builder -- cargo run --quiet -- up --manifest openclaw --name oc`
- `limactl shell mvm-builder -- cargo run --quiet -- logs oc`
- `limactl shell mvm-builder -- cargo run --quiet -- down oc`
- `limactl shell mvm-builder -- nix build .#packages.aarch64-linux.default`
- `limactl shell mvm-builder -- nix path-info -rsh /nix/store/<hash>`
- `limactl shell mvm-builder -- cargo test --workspace`
- `limactl shell mvm-builder -- cargo clippy --workspace -- -D warnings`
- `limactl shell mvm-builder -- cargo check --workspace`

**Important:** `mvmctl` (via `cargo run`) commands like `build`, `up`, `down`, `logs`, and `ls` must be run inside the Lima VM — they talk to Firecracker which only runs inside Linux. `cargo test` / `cargo check` / `cargo build` should run on the macOS host by default (see "Run cargo on the macOS host" above); only `cargo clippy --workspace --all-targets` and tests gated on `target_os = "linux"` need Lima. `cargo run -- dev` always runs on the macOS host directly.

## Worktree Workflow for Features

Every feature, refactor, or non-trivial bug fix is developed in a git worktree — code edits and cargo invocations happen inside the worktree directory. Git operations (status, add, commit, stash, rebase, push, fetch, pull, hook execution) happen from the main `mvm/` checkout, with `-C` pointing at the worktree when needed. The main checkout is the single git operator; worktree directories are code+build sandboxes only.

### Never commit directly to `main`

`main` is updated only via merged pull requests — never by `git commit` against the local `main` branch, even from the main checkout, even for docs-only changes, even with `--no-verify`. Reasons:

- **Safety against parallel agents.** Multiple agents share `.git/`. Any agent that pulls/rebases/`reset --hard origin/main` (a routine recovery move) silently discards local-only commits on main. Branches that exist on `origin` cannot be wiped this way.
- **CI gating.** The full clippy + nextest + supply-chain + flake-check matrix only runs on PRs. A direct commit ships untested.
- **Audit trail.** PR descriptions, CI status, review comments, and merge events form the project's history. A local commit pushed to main loses all of it.

If you have changes intended for `main`, push them to a branch and open a PR — even a one-line typo fix. The repo's GitHub settings are not branch-protected, so the convention is the only thing keeping main clean.

The only `git` commands that should ever target `main` directly are read-only (`git log main`, `git show main:path`) or routine sync (`git fetch origin`, `git pull --ff-only origin main`).

### Creating the worktree


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinylabscom/mvm](https://github.com/tinylabscom/mvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
