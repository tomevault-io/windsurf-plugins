---
trigger: always_on
description: **Nanolathe** is an MIT-licensed reimplementation of the Total Annihilation
---

# Nanolathe — Agent Instructions

**Nanolathe** is an MIT-licensed reimplementation of the Total Annihilation
engine. Behavior comes from clean-room analysis of retail `TotalA.exe`; content
comes from the original assets. Everything is data-driven—never invent what the
executable or assets already define.

---

## The four rules

These override plans, work units, and agent judgement.

**1. Never invent behavior.** For unresolved questions, leave
`TODO(question): <unknown, and what would settle it>` at the code site, record
the gap in the owning research doc, and report it. An honest gap is complete;
a plausible guess is a defect. Verify any **Supported inference** before work
depends on it—inferences here have been found inverted.

**2. Never undo another agent's work.** Never reset, rebase, revert, amend, or
force-push `main`; never discard, stash, or overwrite another agent's changes.
Fix forward with an explanatory commit. Leave foreign dirty work alone and
report it; treat files you did not write this session as live concurrent work.

**3. Clone behavior, not code.** Raw disassembly, decompiler output, addresses,
register traces, and generated names stay in `$HOME/ta-decompile`. Translate
that analysis into an independently worded description of what the algorithm
does before anything enters `research/`, code comments, or commits.

**4. Everyone works in a worktree.** Never edit the shared `main` checkout.

### Intentional Modern gameplay

The user explicitly authorizes **Modern** gameplay (default) alongside opt-in
**Strict 3.1**. Retail research defines the strict baseline. Approved Modern
rules are intentional departures, not parity defects: **do not remove them
merely because retail behaves differently**.

Every new intentional gameplay departure must be selected through the existing
central `gameplay.Mode`, disabled by Strict 3.1, and documented as **Nanolathe
Modern policy** in the owning design document. Record the strict behavior,
modern behavior, boundaries and tests there; do not rewrite retail research to
claim the new policy is historical behavior. Tests must preserve both the
Modern contract and the Strict bypass, including RNG and resource effects.
This is an explicit exception to rule 1 for approved policy, not permission to
invent unresolved retail mechanics. Renderer and host preferences retain their
separate controls.

Use the existing interfaces and `session.RuleSet` registry described in
[DESIGN_GAMEPLAY_RULES](docs/DESIGN_GAMEPLAY_RULES.md), especially §9, for
future gameplay work. Extend the owning interface and both reserved defaults
when a new decision is needed; justify a new owning-package seam only when
none fits, and compose it through the same `RuleSet`. Do not add a second
registry, capability-selection system, or scattered gameplay booleans.
Load-time content profiles remain separate from gameplay selection. Research
an extension before proposing its contract; evidence that a patch implements
a behavior is not authorization to enable that behavior in Nanolathe.

Current policies: [terrain admission](docs/DESIGN_WEAPONS_PROJECTILES.md#231-modern-terrain-admission),
[Hold Fire](docs/DESIGN_UNITS_ORDERS_COB.md#modern-hold-fire), and
[factory-exit yielding](docs/DESIGN_ECONOMY_CONSTRUCTION.md#modern-factory-exit-yielding), and
[construction-site clearance](docs/DESIGN_ECONOMY_CONSTRUCTION.md#modern-construction-site-yielding), and
[authored build membership](docs/DESIGN_ECONOMY_CONSTRUCTION.md#modern-authored-build-membership).
See also [INVARIANTS.md I11](docs/INVARIANTS.md#i11--retail-baseline-and-modern-gameplay).

---

## Clean-room discipline

Applies to everything committed: `research/`, code comments, commit messages,
test names, reports.

**Never commit:** executable addresses/offsets; decompiler-generated names;
disassembly or decompiler output; register narration; executable structure
layouts.

**Do write:** implementable plain-language algorithms and arithmetic; named
concepts rather than addresses; file offsets only as authored format layouts in
`research/formats`; and **Established**, **Supported inference**, or **Unknown**
confidence for every claim.

If a behavior cannot be described without an address, analysis is unfinished.
Keep the address trail in `$HOME/ta-decompile/notes/` for reproducibility.

---

## Research

`research/` is a curated reference, not a notebook:

```
research/
  formats/                  one doc per file format — byte layouts, defaults,
                            conversions. Source of truth for HOW TO READ BYTES.
  extensions/               non-retail extension contracts and evidence policy;
                            see extensions/README.md. Never retail evidence.
  retail-executable-spec/   behavioral contracts. Source of truth for WHAT
                            RETAIL DOES.
    README.md               index, reading order, evidence language
    01..08-*.md             eight category docs, each owning one feature area
                            exhaustively: 01 runtime/determinism, 02 content/
                            vfs/formats, 03 world/visibility/rendering/audio,
                            04 units/orders/scripts/movement, 05 economy/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanolathe-gg/nanolathe](https://github.com/nanolathe-gg/nanolathe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
