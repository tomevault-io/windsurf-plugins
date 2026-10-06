---
trigger: always_on
description: VERA20k is a cross-platform Rust reimplementation of the Command & Conquer:
---

# VERA20k — Project contract

VERA20k is a cross-platform Rust reimplementation of the Command & Conquer:
Yuri's Revenge engine (`gamemd.exe`), using original retail rules and assets.
The goal is to reproduce native gameplay behavior through a Rust-native architecture,
while scaling to **20,000 units and 30 players**. VERA20k must build and run on Linux,
macOS and Windows; keep architecture and dependencies compatible.

All simulation-affecting math must produce identical results across supported
platforms and CPU architectures for identical state, inputs and RNG. Prefer
`SimFixed` for simulation calculations; document and validate any differences
from native precision and rounding.

Development follows complete gameplay mechanisms and their required dependencies,
with clear ownership of state and shared logic. Native executable behavior and
retail data establish what the implementation must reproduce.

This contract governs Codex and Claude. Use engineering judgment; skills are optional
specialized help. Specific user instructions override workflow defaults.

## Intent and autonomy

The user often thinks aloud. Follow intent, keep exact technical values literal,
and treat proposed causes as hypotheses. Investigate contradictory observations.
Push back briefly with evidence; respect informed decisions.

Complete authorized work without routine approval. Resolve discoverable questions;
ask only for missing authority or consequential, undiscoverable preferences.
Research/review alone does not authorize implementation. Preserve scope amendments
and stop instructions across continuations.

Be brief, plain and result-first.

## Exactness and evidence

Confirm gamemd-derived changes against original instructions, active callers, and
retail data. Ghidra annotations are often wrong; treat them as
leads, not proof. Confirm active-YR reachability; unreachable claims need a breakpoint or
flag-to-leaf trace. Never invent offsets, identities or behavior.

Priority follows player visibility and frequency; it does not establish equivalence.
Differences a player cannot see, such as 1–2 px offsets or small color deltas, get a
one-line residual, not investigation; when a visual detail matters, port the drawing
code instead of fitting captures.
Missing or unproven required behavior keeps an exhaustive task open.

Distinguish and cite:

- **Native behavior established:** body/caller/data evidence.
- **Rust regression tested:** named implementation checks.
- **Parity demonstrated:** gamemd-derived executable comparison or exhaustive proof,
  bounded by stated coverage. A sample cannot certify the whole mechanism.

Parity goldens come from native execution/emulation, capture or retail bytes,
not hand calculations or prior Rust; inputs a comparison takes from VERA need
their own native evidence. Avoid unqualified “VERIFIED”/“complete”.

Arithmetic, rounding, RNG draws (count, order, stream) and timer cadences derived from
reading are uncertain until executed. Before merging a mechanism, compare them against
native execution ([Unicorn](tools/native_oracle.md) or capture) and pin the results as
golden values in its Rust tests. Control flow and ordering may rest on instruction-level
reading. Each PR states the evidence level each claim reached.

Preserve native comparisons as reproducible harnesses and results, recording binary
identity and coverage limits. Link them to Rust tests where practical; parity claims
must cite saved evidence and actual validation results.

Each cohesive gamemd-derived Rust behavior carries nearby native identity/address
and source; sim-behavior commits cite their evidence.
Consult the [Ghidra reference](docs/research/ghidra-workflow.md) for access,
interpretation pitfalls and shared-database edits.

## Retail data

gamemd behavior is native code plus the INI keys it reads; port both. For each key,
port the native read: file and layer, section, exact-case key, default, parser, clamps,
read order and any post-read pass. Defaults come from the constructor or the reader's
default argument, never from retail values. Take type names and tuning values from the
keys gamemd reads; hard-code a name only where gamemd holds the literal. Parse with the
native readers in `src/rules/ini_value.rs`, not `str::parse`.

Read retail values in every applicable layer before choosing constants, priorities or
fixtures. Each scenario re-reads rules from `RULESMD.INI`, `LANGRULE.INI` when present,
the selected mode INI, then the map (campaigns and one mode add passes); the map also
extends `AIMD.INI`. `ARTMD.INI` and the sound, EVA and theme INIs are not layered, and
gamemd never reads the RA2 base INIs in `ini/`. A branch gated on a key the retail rules
leave unset is dormant, not unreachable. Check retail data with a gamemd-exact parse,
not grep. When INI values decide an outcome, test through the production reader on
retail INIs. Details: [retail INI reference](src/rules/mod.rs).

## Architecture and delivery

Use the simplest implementation that fully satisfies the required behavior. Avoid
unnecessary abstractions, duplicated logic and speculative features; prefer clarity
over minimizing line count.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuriPlanet/vera20k](https://github.com/YuriPlanet/vera20k) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
