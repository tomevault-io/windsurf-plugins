---
trigger: always_on
description: Generates MTXML files (ApplicationProgram1.mtxml, Hardware1.mtxml, Catalog1.mtxml) from the demo System B device definition. Use `--knxprod` to also generate a signed `.knxprod` package.
---

# zweidraehte

## Mission

We are building a KNX device stack in Rust targeting both embedded
devices in a no_std, no-alloc environment and embedded Linux userspace
systems (and possibly more later).

The stack needs to be conformance compliant and generic enough so that
we can replace different layers and servers in the stack for different
use cases when building devices. It's best to stick to existing
patterns where applicable.

We are also working on a product definition XML generator: device
definitions in Rust macros (parameters, communication objects, dynamic
ETS pages) from which we generate the MTXML/`.knxprod` files that ETS
imports — one source of truth for firmware behaviour and the product
database entry. See "Product definition generator" below.

## Conformance testing

Run the conformance tests with `cargo run --bin conformance-runner`.
Pass a test name, suite name, or a substring of either as the first
argument to run a subset. Do not truncate the output of a test run —
it can be long. The tests take a long while; if you need to inspect
the output, pipe it into a file once and grep that file instead of
re-running the suite. Before running, rebuild the DUT binaries with
`cargo build` — the runner spawns them as separate executables.

## Authoring conformance tests

### Suite-level preparation and teardown

`TestSuite` has two hooks for managing global DUT state around a suite:

- `.with_preparation(vec![...])` — steps run once before any test case
  in the suite. Use for non-trivial setup that all cases depend on
  (e.g. loading Security IO, seeding SIAT entries, initial SyncReq).
  If preparation fails, **all tests in the suite are skipped** — the
  runner reports "Preparation failed - skipping suite tests". A
  cascade of skipped suites usually traces back to a missing or
  misordered preparation step.
- `.with_teardown(vec![...])` — steps run once after all cases finish
  (pass or fail). Use to restore global DUT state so the next suite
  starts from a known baseline. Teardown failures are logged but do
  not affect the suite's pass/fail count.

When a test case mutates global DUT state, you have two choices:

1. **Self-contained**: undo the mutation at the end of the case so
   other cases (and suites) are unaffected. Pattern used in e.g.
   3.8.10.1 restoring the GK table entry.
2. **Suite teardown**: if the mutation can't be cleanly reverted per
   case (factory reset, tool key rotation under multiple branches,
   sync rate-limit consumption) put the restore in `with_teardown`.

Destructive operations that leave the DUT in a state the next suite
can't recover from (wiped address / association / group-key tables,
missing IA, etc.) should use `full_reset(timeout_ms)` in the suite
teardown. `TestStep::FullReset` kills the DUT, rewrites shared memory
with the factory-default snapshot, zeroes the sequence-number tail
region (so the respawned DUT starts with fresh seq counters rather
than inheriting stale ones from the previous run), respawns, and
drains ROI frames. Pair it with `wait(1500)` when the prior test
consumed a `S-A_Sync_Req` slot — the DUT's sync rate-limit timer is
wall-clock and survives the respawn.

### Timing, fast mode, and rate limits

The runner compresses inter-step waits by a default 50× via the
`KNX_TIME_DIVISOR` env var (the `--realtime` flag disables it).
`wait(ms)` calls scale accordingly.

DUT-side timing windows that tests depend on (TL ACK / connection
timeouts, `S-A_Sync_Req` rate limit) are scaled by the same divisor
inside the DUT when built with the `conformance` feature, so logical
ordering is preserved without burning real wall-clock. The sync
rate-limit window is 1 s per spec and scales to ~20 ms in fast mode;
tests between back-to-back syncs park `wait(1500)` which comfortably
clears the window in both fast and realtime runs.

## Product definition generator & MDT replication

To prove the ETS DSL has feature parity with real products, we
replicate an existing MDT device
(`examples/devices/src/mdt_push_button_lite.rs`): the parameters, the
enums, the comm objects, and the dynamic pages that select different
combinations of references and show/hide parameters and/or
communication objects based on the currently selected configuration.
After parity, we optimize the DSL for readability and quality of life.
Module definitions (reusable multi-channel blocks) are replicated
conceptually with a small test device
(`examples/devices/src/module_test_device.rs`).

**The manufacturer reference material is NOT in the repository.** The
vendor XML lives in a git-ignored local directory `manuf_tool_data/`
that only exists on machines with a private copy (licensed vendor
data). If present, the relevant files are:

- `manuf_tool_data/MDT_KP_BE_01_Push_Button_Lite_55_63_V14/M-0083/M-0083_A-009B-14-E59D.xml`
  — the MDT Push Button Lite reference that `mdt_push_button_lite.rs`
  replicates.
- `manuf_tool_data/VC-EASY-03_MDT_KP_V35/M-0083/M-0083_A-0070-35-1740.xml`
  — contains the module definitions the module test device mirrors.
- `manuf_tool_data/knx_project.xsd` — XSD schema for validating the
  XML.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ntrnnr/zweidraehte](https://github.com/ntrnnr/zweidraehte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
