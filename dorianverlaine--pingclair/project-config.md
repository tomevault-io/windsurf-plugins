---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Read these first

`AGENTS.md` is the operating manual and takes precedence over this file wherever
they overlap. It covers the ghost-process trap, editing discipline (comment
style, emoji conventions, commit subjects), architecture constraints per
subsystem, and documentation ownership. This file adds the command details and
the cross-crate picture that only emerge from reading several files at once.

These documents own distinct things, and the project treats mixing them as a
defect:

| Document | Owns |
| --- | --- |
| GitHub issues | Everything outstanding: the plan, defects found and left alone, compatibility gaps. One item, one issue, one place. Templates and labels are in `CONTRIBUTING.md`; `Closes #123` in a PR is what closes the item. **Replaced `docs/TODO.md` and `TRIAGE.md` in August 2026** — a stale copy of either in a checkout is history, not the answer. |
| `docs/STATUS.md` | Which public claim has evidence behind it, and where. Three levels: code exists, local tests pass, verified on clean Linux. 🔒 Local only. |
| `docs/guardrails/{testing,config,tls,proxy}.md` | Environment constraints and implementation rules, one file per subsystem. Every entry is a failure that already happened. `docs/GUARDRAILS.md` is the index over them, nothing more. |
| `benchmarks/README.md` | Performance claims and methodology. Raw per-run evidence stays local under `benchmarks/results/`, never committed. |
| `CHANGELOG.md` | What changed between releases, for someone upgrading. Written the same day as the change. |

Two more are 🔒 local reference rather than working documents:
`docs/CADDYFILE_COMPATIBILITY_MASTER.md` answers "does Pingclair support this
Caddy directive"; the other `docs/CADDYFILE_*.md` files are frozen 2026-08-01
audit records, deliberately excluded from `documentation.rs` because they are
full of configurations that must *not* compile. Do not read any of them as
current behavior — check the code.

Implemented is not verified. The verification ledger deliberately separates
"code exists", "local tests pass", and "verified on Linux/VPS"; never promote
an item between those without evidence under
`benchmarks/results/<date>_<commit>/` (kept locally, not committed).

When the maintainer planning files are present, run
`scripts/snapshot-sensitive-plans.sh start` before and `end` after a session;
a snapshot validation failure blocks handoff.

## Commands

The canonical gate is `just ci` — fmt-check, clippy, cargo-shear, repo-lint,
docs-lint, the full nextest suite, and bench smoke — and CI runs the same
recipes. **`+1.97.1` is not decoration**: the workspace declares
`rust-version = "1.97"` and CI pins 1.97.1. A different local compiler —
newer or older — has different inference and rustfmt line breaking;
all-green locally followed by all-red in CI has already happened in both
directions (newer-than-CI on 2026-07-29, an older toolchain in the release
image on 2026-08-02).

```bash
just ci
```

Narrower runs:

```bash
just test -p pingclair-proxy                               # one crate
cargo +1.97.1 nextest run -p pingclair --test integration --no-fail-fast
cargo +1.97.1 nextest run -p pingclair --test integration test_name -- --nocapture
cargo +1.97.1 nextest run -p pingclair-proxy --test h3_end_to_end --no-fail-fast
```

`pingclair/tests/integration.rs` spawns the real compiled binary and makes real
localhost requests. It is the main end-to-end gate, not a mocked test — so a
stale listener on a test port produces misleading failures. Read the
ghost-process section of `AGENTS.md` before debugging a suspicious localhost
failure.

Some integration tests are load-sensitive rather than flaky in isolation.
Reproduce with several concurrent full suites, not repeated single runs:

```bash
cargo +1.97.1 build --tests -p pingclair
BIN=$(find target/debug/deps -maxdepth 1 -name 'integration-*' -type f -perm -u+x ! -name '*.d' -exec ls -t {} + | head -1)
for i in $(seq 1 6); do "$BIN" > /tmp/full_$i.log 2>&1 & done; wait
```

### H3 verification

macOS unit tests do not validate linking or QUIC behavior. After any change to
H3 or the TLS dependency tree, run `just h3` (the three maintained scripts)
and the Linux half:

```bash
scripts/test-h3-day28-local.sh              # SNI, Alt-Svc, body sizes, POST, 413, keepalive
scripts/test-h3-cancellation-local.sh       # SSE, downstream cancellation, trailer rejection
scripts/test-h3-client-auth-local.sh        # mutual TLS, and the SNI/:authority rule that protects it
```

Both need a curl built with HTTP/3 (`brew install curl` provides one; the system
curl does not). CI runs the Linux half post-merge on `ubuntu-24.04`; a manual
Linux box can use `rust:1.97-bookworm`, which needs `cmake` for BoringSSL and
`clang`/`libclang-dev` for bindgen — without them `boring-sys` fails in its
build script.

macOS has a system proxy on `127.0.0.1:1082`. Reqwest test clients must use
`.no_proxy()`; curl needs `--noproxy '*'`.

## CI (two-layer)

The merge gate is `blocking-ci.yml`: it runs the fast `rust-ci` (path-aware
`just ci` plus the known-flaky retry policy), the Docker image build and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorianverlaine/pingclair](https://github.com/dorianverlaine/pingclair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
