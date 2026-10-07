---
trigger: always_on
description: Working notes for AI agents in this repository. `README.md` and `CONTRIBUTING.md` are the
---

# AGENTS.md

Working notes for AI agents in this repository. `README.md` and `CONTRIBUTING.md` are the
human-facing documents; this is the short version to read before your first edit.

## What this is

EdgeLinkd is a Node-RED compatible flow runtime written in Rust with the Node-RED web
editor built in. Rust workspace layout:

| Path | Contents |
|---|---|
| `src/` | the `edgelinkd` CLI binary (`run`, `list`, web UI / headless) |
| `crates/core/` | the runtime: engine, flows, nodes, context, message model, JS bridge |
| `crates/web/` | admin API + web UI server (axum) |
| `crates/macro/` | `#[flow_node]` / `#[global_node]` proc macros (self-registration) |
| `crates/pymod/` | `edgelink_pymod`, the Python extension the test suite drives |
| `node-plugins/` | statically linked node plug-ins |
| `tests/` | pytest port of Node-RED's mocha spec suite |
| `scripts/` | build/packaging helpers + the spec coverage audit |
| `3rd-party/node-red/` | pinned Node-RED checkout (git submodule, v4.0.9) — the behavioural reference |

## Design philosophy: embedded-first, compatible where supported

EdgeLinkd targets **embedded and resource-constrained deployments**, not a desktop Node.js
runtime. It is a Node-RED *compatible* runtime, not a Node-RED clone: reproducing every
Node-RED feature one-to-one is **not** a goal.

The compatibility contract is therefore conditional, and it cuts both ways:

- **What we ship does behave like Node-RED.** For every node, option and property type we
  offer, the observable behaviour must match the pinned upstream v4.0.9 — message semantics,
  error and status behaviour, editor contract — and that is what the ported spec tests in
  `tests/` assert.
- **What does not fit the budget, we do not offer.** A feature whose memory or binary-size
  cost is too high for the target hardware, or that depends on the Node.js ecosystem, is out
  of scope and may stay unimplemented indefinitely. That is a design decision, not a bug and
  not a TODO waiting to be picked up.

Two consequences for day-to-day work:

1. **The Node-RED editor *is* the EdgeLinkd UI**, and users design their flows directly
   against EdgeLinkd — the workflow this project optimises for is authoring in the editor and
   running immediately, not "design in Node-RED, test there, then copy `flows.json` over".
   Anything the editor can produce must therefore either work, or fail loudly and at once
   (deploy error, node error/status, `EdgelinkError::NotSupported`). It must never be silently
   ignored or silently reinterpreted.
2. **Never fake support.** No `todo!()`, no silent no-op, no stub returning a plausible
   value, no option that is accepted and then ignored. A half-working feature that looks fine
   is worse than an absent one, because the user cannot tell the difference.

Where scope decisions are declared:

| Place | What it records |
|---|---|
| `README.md` roadmap | feature-level ✅/⬜ status |
| `tests/REDNODES-SPECS-DIFF.md` (generated) | per-node spec coverage against upstream |
| `@pytest.mark.skip(reason=...)` in `tests/` | a spec we deliberately do not support |
| `EdgelinkError::NotSupported` at runtime | configuration that is recognised but out of scope |

## Commands

| Task | Command |
|---|---|
| Build everything (including the Python extension) | `cargo build --all` |
| Rust tests | `cargo test -p edgelink-core` while iterating, `cargo test --workspace --features full` for the full set |
| Node-RED spec tests (pytest) | `pytest ./tests -v` — needs `cargo build --all` first |
| Format check (CI gate) | `cargo fmt --check` |
| Lint (CI gate) | `cargo clippy --all-features --tests --all` |
| Spec coverage report | `python scripts/specs_diff.py 3rd-party/node-red -o tests/REDNODES-SPECS-DIFF.md` |

CI (`.github/workflows/CICD.yml`): `fmt-and-check` (fmt + `cargo check --workspace`) runs on
every push; the Linux job additionally builds, runs the Rust tests and `pytest ./tests -v`;
clippy runs for master-bound PRs; Windows/ARM jobs run on schedule/dispatch. Keep all of
them green.

The spec coverage report is an **audit of what we support, not a completion target**: 100%
parity with the upstream Node-RED suite is explicitly not a goal (see the design philosophy
above). Read its `:x:` rows as "upstream tests for features we do not offer yet", and treat a
missing `it()` for an out-of-scope feature as a decision to record, not work to schedule.

## Rules

1. **A supported node is not done without its spec tests.** Implement it in Rust *and* port
   the upstream Node-RED `it()` tests to pytest, then register the pair in
   `scripts/specs_diff.json`. Start from the `port-node-red-node` skill
   (`.agents/skills/port-node-red-node/SKILL.md`) — it carries the node template, the test
   harness API, the audit workflow and the verification steps. Port the specs for what you
   support; do not port specs for out-of-scope behaviour just to lift the numbers.
2. **Test titles are a contract.** `@pytest.mark.describe` / `@pytest.mark.it` texts must
   match the upstream JS `describe()` / `it()` titles character for character;
   `scripts/specs_diff.py` diffs them and reports drift as a `-`/`+` pair. Upstream tests we

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oldrev/edgelinkd](https://github.com/oldrev/edgelinkd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
