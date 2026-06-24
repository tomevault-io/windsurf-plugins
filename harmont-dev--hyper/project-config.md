---
trigger: always_on
description: Guidance for AI agents (Claude Code and others) working in this repo. Human
---

# AGENTS.md

Guidance for AI agents (Claude Code and others) working in this repo. Human
contributors: see `CONTRIBUTING.md` — this file does not replace it, it sharpens
the parts agents get wrong.

## What Hyper is

A distributed orchestrator for Firecracker microVMs, written on the BEAM
(Elixir `~> 1.20`, OTP 28+). A small privileged Rust setuid helper
(`native/suidhelper/`, crate `hyper_suidhelper`) performs the Linux operations
the BEAM cannot do safely (losetup, dmsetup, chroot jails, device nodes).
Postgres is the only external runtime dependency (the image database).

## Commands

```sh
mix check        # THE gate. Must pass before any PR. Runs, in order:
                 #   format --check-formatted   (formatting is not optional)
                 #   compile --warnings-as-errors --force
                 #   credo --strict
                 #   test --warnings-as-errors
                 #   dialyzer                    (strict; @specs required)
mix test                              # Elixir suite (needs Postgres for DB tests)
mix test test/unit test/controls      # pure tests, no Postgres/Firecracker needed
cargo nextest run                     # Rust suite (run inside native/suidhelper/)
```

Pure tests under `test/unit` and `test/controls` need neither Postgres nor
Firecracker. DB-backed tests need `mix ecto.create && mix ecto.migrate` first.

## Layout

- `lib/` — Elixir source. Tests in `test/` mirror this tree.
- `lib/unit/`, `lib/controls/`, `lib/sys/linux/proc/`, `lib/hyper/redist/` —
  **pure cores**: units algebra, EWMA/rate controls, `/proc` parsers, hashing.
  No processes, no I/O. These are where property tests pay off most.
- `native/suidhelper/` — the privileged Rust helper. Source in `src/`, tests in
  `tests/` (see Rust rules below).
- Generated, do not hand-edit: `lib/hyper/firecracker/api/{operations,schemas}`
  (regen `mix firecracker.gen`) and `lib/hyper/grpc/v0/hyper.pb.ex`
  (regen `mix grpc.gen`). Both are gitignored and rebuilt by a Mix compiler.

## Testing philosophy — read this before writing any test

**A good test proves the spec, not that the code ran.** Writing a passing test
is not the goal; writing a test that would *fail if the behavior were wrong* is.
Before adding a test, ask: "what does this module promise, and what input could
break that promise?" If a test cannot fail for any realistic implementation bug,
it proves nothing — delete it.

### Prefer property-based tests. Find the invariants.

StreamData (Elixir) and proptest (Rust) are already wired in. For any pure
function — parsers, codecs, algebra, validators, hashing, scheduling math —
**reach for a property test first**. The work is identifying the invariant. Hunt
for these families (the existing suites are worked examples of each):

- **Round-trip / inverse** — `parse(render(x)) == x`, `decode(encode(x)) == x`,
  `with_value(value(q)) == q`. See `test/unit/quantity_properties_test.exs`,
  `test/sys/linux/proc/stat_properties_test.exs`,
  `native/suidhelper/tests/util/safe_path.rs` (`relative_to` reconstructs).
- **Algebraic laws** — commutativity, associativity, identity, inverse, total
  order. See the additive-group laws in `quantity_properties_test.exs`.
- **Oracle / model** — the result equals an independent reference computation
  (`CpuTimes.total == Enum.sum(cols)`).
- **Invariant preserved** — a property that holds for *every* output regardless
  of input (a confined path never contains `..`; a refcount is never negative).
- **Idempotence / metamorphic** — `f(f(x)) == f(x)`; or a known input change
  produces a known output change.
- **Error & refusal contracts** — invalid input *always* raises/returns the
  specific error, and is never silently accepted. This is a property too: see
  "mixing two dimensions always raises" and `rejects_any_loose_component`.
  For security-sensitive code (the setuid helper, path confinement) the refusal
  property is the *most* important test in the file.

State the laws under test in the module's `@moduledoc` (or a Rust `//!` doc),
the way the existing property suites do — it forces you to name the contract.

### When an example test is the right tool

Property tests are not a religion. Use a plain example/smoke test when:

- the behavior has no meaningful input space to generate over (a specific
  parse of one real `/proc/meminfo` fixture, one gRPC round-trip);
- you are pinning one concrete edge case or regression;
- generating valid inputs would be more code (and more bugs) than the thing
  under test.

A few good examples that exercise real logic beat a generator that only ever
hits the happy path. Pair them: properties for the laws, examples for the
representative cases and the nasty edges.

### Do not write slop

These will be asked to be removed (per `CONTRIBUTING.md`):

- one-assertion-per-getter / setter tests that restate the struct definition;
- tests asserting on mocks you set up in the same test (proves the mock, not
  the code);
- tautologies — `assert f(x) == f(x)`, or recomputing the implementation inside
  the assertion;
- snapshot/coverage-padding tests with no invariant behind them;
- a `property` block whose generator is so narrow it only emits one value.

Coverage is a side effect of good tests, never the target.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harmont-dev/hyper](https://github.com/harmont-dev/hyper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
