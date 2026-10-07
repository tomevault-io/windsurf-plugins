---
trigger: always_on
description: This file is not version controlled. Do not reference it in commit
---

# plushie-elixir

This file is not version controlled. Do not reference it in commit
messages, pull requests, or documentation.

Native desktop GUI framework for Elixir, powered by iced. Implements
the Elm architecture (init/update/view) with commands and subscriptions.
Communicates with the Rust binary over stdin/stdout using MessagePack
(default) or JSONL.

## Stewardship

Direction, trust posture, goals, and explicit non-goals are captured
in `docs/stewardship/`. That directory is the authority on what work
the project takes on and what it declines. The summary below is enough
for routine work; pull the relevant doc when an axis is in play. Use
`docs/stewardship/triage.md` as the routing tool when the answer is
not self-evident.

Pre-1.0: no backcompat, right design wins, rename across SDKs is fine.
Post-1.0: stability obligations begin (Hyrum's Law). plushie-rust =
protocol authority. plushie-elixir = canonical API-shape reference
(rename here = six-SDK change). Cross-SDK parity audited in sibling
`plushie-sdk-parity/`. Hex package is a library, not an OTP application.

### Disciplines (non-negotiable)

Tests through real renderer; cross-SDK claims verified by reading
source on each side; design before code at boundaries (public API,
DSL surface, wire codec, type behaviour); clarity is the bar; no
half-built features; local cleanup not scope creep; no legacy shims
pre-1.0.

### Goals

Wire codec fidelity on host side; cross-SDK concept parity (semantics
converge, syntax diverges per language); Elm-architecture purity
(`init/update/view`, return-shape validation, commands as pure data,
pure view, declarative subs); lightweight runtime (no idle work, no
polling, minimal tree diff via LIS); fault tolerance (renderer crash
auto-recovers + state re-syncs, app exception reverts to last good
model, view-error tracking with frozen-UI overlay, neither side takes
the other down); macro DSL clarity.

### Non-goals (declined, not deprioritized)

Backcompat before 1.0; per-Elixir API ergonomics that diverge from
cross-SDK shape; API stability hardening pre-1.0 (single 1.0 sweep,
not piecemeal); coverage targets as a metric; mocking renderer for
speed; micro-optimization at cost of readability; refactoring without
a forcing function; DSL extensions for hypothetical future widgets;
defending against speculative deployment shapes.

### Trust model

Asymmetric. Renderer-to-host = closed and typed; host structurally
protected today (typed event decoding, no opaque-blob path, effect/
query response correlation by wire ID, no host-side eval, atoms only
from closed enumeration). Host-to-renderer = broad by design (file
paths, fonts, images, screenshots, effects, structured renderer exec
args); bounding it is the capability-manifest roadmap in plushie-rust.
Wire = byte-stream agnostic; confidentiality + integrity delegated to
outer transport. Same-access (user attacking themselves) out of scope.

### Resilience

Things-go-wrong axis, not adversary axis. App exception revert in
`init/update/view`; view-error tracking with frozen-UI overlay after
warn threshold; renderer crash auto-recovery with fresh snapshot
re-sync; rest_for_one supervision (Bridge crash -> Runtime restart;
Runtime crash alone -> Runtime re-syncs to running Bridge); defensive
parsing on the wire (reject + structured error); return-shape
validation in `unwrap_result/1` raises immediately; subscription
failure isolated. Fail-fast on programming-error invariant violations
and unrecoverable bridge startup. Degrade gracefully on user-facing
input. Log suppression after 100 consecutive errors.

### Performance

Lightweight = baseline, not optimization-after-fact. Don't do
unnecessary work in the first place; cost compounds. Worth doing
without benchmark (readability preserved/improved): consolidate
redundant traversals, right data structure, avoid unnecessary `Enum`
passes, move per-frame work that doesn't depend on per-frame inputs
to the edge. Need benchmark first (readability cost real): clever
encoding, big-O without realistic N, ETS over GenServer state without
measured contention, optimization on idle paths. Numeric direction:
16.67ms frame budget at a few hundred to ~1000 nodes; idle CPU = no
measurable work; tree diff is the load-bearing piece (LIS-based child
reorder, memo cache, widget view cache).

### Test discipline

Integration spine: tests exercise real renderer (default `:mock`
backend = real binary, real wire, real Core, no GPU). Three modes
(cross-SDK contract): mock (default, fastest), headless (tiny-skia,
pixels), windowed (full iced, real display). Pooled mock backend
multiplexes via `--max-sessions N`. Stubs acceptable only for forced
crash sim, malformed wire bytes, direct `update/2` shape tests, test
infra. Sync via `Runtime.sync/1`/`get_model`/`get_tree`, never `:sys`.
Tests as documentation; slow tests = slow code; failing test before
fix. Test apps must return `window` nodes from `view/1`. Capture logs
on intentional-error tests with `@describetag capture_log: true`.

### Simplicity

Clarity = constraint, not aspiration. Reader-cost compounds.
Readability wins ties. Abstraction earns its place: 3 similar lines
> premature abstraction; 3rd use earns consideration not commitment;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plushie-ui/plushie-elixir](https://github.com/plushie-ui/plushie-elixir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
