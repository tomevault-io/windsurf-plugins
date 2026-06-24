---
trigger: always_on
description: Before committing any Rust change, **always** run:
---

# agend-terminal — Claude working notes

## Rust workflow

Before committing any Rust change, **always** run:

```bash
cargo fmt
cargo clippy --all-targets -- -D warnings
```

CI runs these in the first two steps of `ci.yml`. Skipping them locally means
the next push fails and needs an extra "fix fmt / fix clippy" round trip.

### Before `git push`: run the full CI-parity preflight

```bash
scripts/preflight.sh          # full matrix; --quick skips the Windows check
```

This is the one-shot mirror of CI's `check` job and the best way to avoid a
local-green → CI-red round trip. It runs `cargo fmt --check`,
`cargo clippy --all-targets --features tray -- -D warnings`,
`cargo test --tests --features tray` (unit + integration + invariants), and a
**Windows cross-check** (`x86_64-pc-windows-msvc`) — the keystone, since
Windows-only code (`libc::getppid`, `/bin/sh` spawns, `UnixStream`) compiles
fine on a unix dev box but breaks CI's `windows-latest` runner.

The Windows step needs the MSVC C toolchain because a transitive C dependency
(`ring`) won't otherwise cross-compile on macOS/Linux. Install it once:

```bash
cargo install cargo-xwin && rustup target add x86_64-pc-windows-msvc
```

Without `cargo-xwin` the Windows step SKIPs with a hint (never false-fails);
CI's `windows-latest` runner stays the backstop. The preflight is intentionally
*not* a git hook — the full matrix takes a few minutes; run it manually.

A pre-commit hook at `.git/hooks/pre-commit` auto-formats staged `.rs` files
and re-stages them. It does NOT run clippy — clippy is too slow for a
pre-commit path. Run clippy yourself before `git push`.

The pre-push hook (`scripts/hooks/pre-push`) runs **two gates**:

1. **CI-parity** (#t-ci-parity-prepush-guard) — on a push whose range touches
   `src/` / `tests/` / `Cargo.*` / `build.rs`, it runs `scripts/preflight.sh
   --quick` (the exact CI `check` commands: `cargo fmt --check`, `cargo clippy
   --all-targets --features tray -- -D warnings`, `cargo test --tests --features
   tray`) and **blocks the push if they fail**. This closes the recurring miss
   where an agent ran only `cargo test --bin` — which SKIPS the `tests/`
   integration targets — declared CI-ready, and CI then rejected it
   (#1734 stale-string integration test, #1735 block_on invariant). Docs-only
   pushes skip the build. `--quick` omits the Windows cross-check (CI's
   `windows-latest` is the backstop for that).
2. **claim-verify** — verifies `Claim:` trailers against the actual diff.

Override either with `git push --no-verify` in emergencies (the daemon-side gate
and CI still apply, so `--no-verify` is not a free pass).

A post-merge hook triggers a background `cargo build --release` when `src/`
files change. Desktop notification on completion. Does not auto-restart the
daemon — operator decides restart timing. Disable with
`git config core.hooksPath /dev/null`.

### If the hook isn't installed

Hooks are per-clone. After a fresh `git clone`, install with:

```bash
scripts/install-hooks.sh   # idempotent — safe to re-run
```

**Fleet-agent coexistence**: the daemon sets each managed worktree's
`core.hooksPath` to `$AGEND_HOME/hooks` (`src/binding.rs::install_hooks`) and
writes only `prepare-commit-msg` there — it does *not* clear the directory. So
`install-hooks.sh` *copies* the tracked `pre-push` into `$AGEND_HOME/hooks/pre-push`,
where it coexists and fires for agent pushes (one copy covers every current and
future worktree, since they share that dir). Agent pushes DO run the hook: the
`agend-git` shim and the `AGEND_GIT_BYPASS` path both `exec` real `git`, which
honours `core.hooksPath`. This coexistence relies on the daemon not deleting
`$AGEND_HOME/hooks/pre-push`; if that changes, re-run `install-hooks.sh`.

## Test fidelity: feed consumers the producer's real output (#1493)

When a test exercises a consumer of some wire format — a parser, a matcher, a
predicate over a string/struct another function builds — **construct the input
by calling the producer, not by hand-writing the shape.** Review check: *"is
this fixture identical to what production actually sends?"*

This is the #1483 false-green class: `notification_is_actionable_wake` matched
bracketed body markers that the real `[AGEND-MSG-PENDING]` pointer never
contains, and the tests hand-crafted that never-emitted shape — so the matcher
was dead code while the tests passed (then drifted again when #1487 added a
`now=` field the crafted strings lacked). Fix pattern: extract one builder
(e.g. `build_pending_pointer`) and route BOTH production and the tests through
it. Hand-crafted input is still correct for testing a parser's *malformed/edge*
handling — just pin the happy-path contract against the real producer.

## sync→async bridge: no raw shared-runtime `block_on` (#1476)

**HARD RULE.** A sync→async bridge MUST NOT call `block_on` directly on a
long-lived shared runtime accessor (`telegram_runtime()`, `discord_runtime()`,
`shared_ci_runtime()`, …). Those are `current_thread` runtimes, so
`<name>_runtime().block_on(...)` panics with *"Cannot start a runtime from
within a runtime"* the moment a caller reaches it from inside a tokio runtime.

This is exactly the bug telegram hit (#1474, teloxide 0.17 made the path

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suzuke/agend-terminal](https://github.com/suzuke/agend-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
