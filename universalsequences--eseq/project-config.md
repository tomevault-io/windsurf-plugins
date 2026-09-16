---
trigger: always_on
description: Prefer `cargo nextest run` over `cargo test`. Verify the prerequisite with
---

# AGENTS.md

## Test selection and runtime policy

Prefer `cargo nextest run` over `cargo test`. Verify the prerequisite with
`cargo nextest --version`; install it with `brew install cargo-nextest` on macOS
or `cargo install cargo-nextest --locked` on Linux. It runs each test in its own
process, which isolates tests that touch shared global state (e.g. the shared
`$TMPDIR/sequencer_dgenlisp` compile output dir), reports per-test wall times,
and schedules the suite better. Fall back to `cargo test` only when installing
nextest is not possible.

Use the narrowest test target that validates the behavior changed. Do not run a
full package or workspace test suite as a default validation or finishing step.

For a unit test in a library, select the exact test:

```sh
cargo nextest run -p <package> -E 'test(=<fully-qualified-test-name>)'
# or: cargo test -p <package> --lib <fully-qualified-test-name> -- --exact
```

For an integration test, select the integration-test binary and the exact test:

```sh
cargo nextest run -p <package> -E 'binary(<test-target>) and test(=<test-name>)'
# or: cargo test -p <package> --test <test-target> <test-name> -- --exact
```

nextest notes: use `--no-capture` where a `cargo test` command would use
`-- --nocapture`; `-E 'test(/regex/)'` selects by regex; nextest does not run
doctests (this repo's tests are all unit/integration tests, so that does not
matter here).

### DGenLisp compiler (fetched, not tracked)

The DGenLisp compiler binary is not in git. `content/dgenlisp.lock` pins the
published distribution per target; run `./scripts/fetch_dgenlisp.sh` once per
fresh checkout (idempotent, sha256-verified) to install it under
`crates/sequencer/tools/` (gitignored). Anything that needs the compiler and
cannot find it hard-fails naming that command. `ESEQ_DGENLISP_TOOL=/abs/path`
overrides it with a locally built compiler.

The compiler is only half of it: it shells out to a hermetic clang/lld stage
pinned in `content/dgen-toolchain.lock` and installed by
`./scripts/fetch_dgen_toolchain.sh`, also once per fresh checkout. That script
fetches targets with a published `url` in the lock; both `arm64-apple-macos`
and `x86_64-unknown-linux-gnu` have one, so a fresh checkout on either
platform bootstraps the stage with no dgen-audio checkout.
`./rebuild_dgenlisp_tool.sh` still vendors the identical mac archive from a
local dgen-audio checkout when you have one; both routes land on the same
pinned sha256.

### Cheap clean-HEAD check

Do not stash and do not cold-clone the repository to determine whether one test
fails at HEAD. Reuse an isolated worktree and a dedicated target directory.
Resolve HEAD once in the working checkout and pin both worktree commands to that
commit: a bare `HEAD` passed to `git -C "$wt"` resolves against the worktree's
own detached HEAD, so a reused worktree would silently stay on whatever commit
it was left at and answer for the wrong tree.

```sh
wt=/tmp/eseq-head-test; target="$HOME/.cache/eseq-head-test-target"
root=$PWD
head=$(git rev-parse HEAD)
[ -e "$wt/.git" ] || git worktree add --detach "$wt" "$head"
git -C "$wt" checkout --detach "$head"
(cd "$wt" && \
  CARGO_TARGET_DIR="$target" \
  ESEQ_DGEN_TOOLCHAIN_ROOT="$root/crates/sequencer/tools/dgen-toolchain" \
  cargo nextest run -p <package> -E 'test(=<fully-qualified-test-name>)')
```

The fetched compiler and the hermetic clang/lld stage are both gitignored, so a
worktree does not inherit either one. `ESEQ_DGEN_TOOLCHAIN_ROOT` (resolved in
the working checkout as `root`, like `head`) points the worktree at the main
checkout's stage; a test that also needs the compiler itself wants
`./scripts/fetch_dgenlisp.sh` run inside the worktree, or `ESEQ_DGENLISP_TOOL`
pointed at the main checkout's binary.

The dedicated target directory keeps Cargo artifacts from the clean checkout
separate from working-checkout artifacts and off `/tmp`, which is a 3.9 GB tmpfs
on the Linux workstation and cannot hold a Cargo target directory. Sharing a
target directory between worktrees can make Cargo run a binary built from the
wrong source tree. The worktree is disposable and isolated, so resetting it
never touches the working checkout.

Budget for the cold build before starting. `-E` filters which tests *run*, not
what gets *built*, so even the narrowest exact test pays for its package's whole
dependency graph: measured 2026-08-24, one `-p sequencer` test took 5m51s and
left 8.1 GB in the dedicated target directory. Still prefer the narrowest exact
test — it saves run time and keeps the output readable — but do not expect it to
save disk. A `--workspace` run costs more of both.

Clean up both directories when they are no longer useful; each is several GB and
easy to forget:

```sh
git worktree remove /tmp/eseq-head-test
rm -rf "$HOME/.cache/eseq-head-test-target"
```

### Test stack budget

`.cargo/config.toml` applies one 16 MiB `RUST_MIN_STACK` budget automatically to
Cargo-launched test processes. The same number is
`sequencer::REQUIRED_THREAD_STACK_SIZE` for explicitly spawned scheduler/UI test
threads. Do not add local 32/64 MiB literals or rely on a remembered shell
prefix.

LLDB investigation for eseq-4tl found that debug overflows while loading the UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [universalsequences/eseq](https://github.com/universalsequences/eseq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
