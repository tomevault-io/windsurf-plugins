---
trigger: always_on
description: This is a coupling framework: components are black boxes with their own state
---

# CLAUDE.md

## Think Before Coding
This is a coupling framework: components are black boxes with their own state
layouts, their own timesteps and their own scientific conventions, and the bugs
that matter live in the seams between them. Don't assume. Don't hide confusion.
Surface tradeoffs.

Before implementing:

 - State your assumptions explicitly. If uncertain, ask.
 - If multiple interpretations exist, present them - don't pick silently.
 - If a simpler approach exists, say so. Push back when warranted.
 - If something is unclear, stop. Name what's confusing. Ask.

Always document these decisions in the comments, and if appropriate in the
documentation (and possibly in the high-level design documentation under
`docs/source/design/`).

Comments should always reference the current state of the code, and explain
*why* it is doing what it is doing, not how it is different to some previous
version of the code (which can get out of date and confusing).

## Finish the Job — No Half Implementations
When asked to fix or implement something, deliver the **complete, faithful**
solution by default — do not ship a partial fix, a band-aid, or a "good enough
for now" workaround and present it as done. In a coupler "faithful" specifically
means the coupled system is right, not just the file you edited: the flux the
exchanger moves has the sign, the units and the grid the receiving component
expects, the carry structure that comes out of a step matches the one that went
in, and conservation across the exchange is checked rather than assumed.

 - If the correct fix turns out to be deeper than expected, do the deeper fix.
   Don't silently descope to the shallow version.
 - A workaround/cap/guard is acceptable **only** as an explicitly-labelled
   stopgap that the user has agreed to — never as a substitute for the real fix.
 - Validate that the full fix actually works (tests + a short coupled run where
   relevant) before calling it done.
 - The only time to stop short is a genuine blocking decision that is the user's
   to make (per "Think Before Coding" above) — surface it and ask. Effort or
   tedium is not such a reason.

## Related findings get rolled into the PR; only unrelated ones become issues
When a piece of work surfaces additional defects or gaps, the default is to
**fix them in the same PR** whenever they are related to the work at hand — same
subsystem, same convention, same failure class, or anything a reviewer would
naturally want to see together. The maintainer would much rather review one
comprehensive PR than a cluster of small follow-ups. The test is: *would the
reviewer be surprised to find this fix in the PR?* If not, roll it in.

File a GitHub issue **only** for findings genuinely unrelated to the current
work — a different subsystem, something needing its own validation campaign, or
something blocked on resources or decisions the current PR cannot wait for. When
an issue is warranted:

 - Title it by the gap (not the PR that found it), with enough context to start
   cold: what is missing, where the hooks already are, and what reference
   behaviour applies.
 - Cross-link it from the code comment or docstring that notes the gap, and from
   the PR.

Either way, a docstring note or PR-body mention alone is never the resting place
for a known defect — it is either fixed in the PR or tracked in an issue.
Deliberately parked/rejected directions don't get an issue — record the decision
and its evidence where the decision was made instead.

## No bespoke run scripts — new configurations go through Hydra
Every runnable configuration must be expressible as a single command with
config-group overrides — never as a standalone driver script:

 - The **target** is `python -m jem.main` with Hydra groups under
   `jem/config/` (atmosphere, ocean, land, seaice, coupling, run,
   experiment), mirroring how `jcm` is driven. That driver and its config
   tree do not exist yet; they are Phase 2 of the API hardening plan (the
   plan itself is not in the repository — `docs/source/design/architecture.md`
   describes the API as it stands).
 - **Until it does exist, do not add new standalone command-line driver
   scripts (a hand-rolled CLI plus a `run.sh`).** A new runnable
   configuration is a notebook or a short snippet in
   the docs that calls the Python API (`Coupler`, the component classes,
   `Coupler.generate_trajectory_function`); it is not a new bespoke driver.
   The bespoke drivers still under `examples/02_experimental/` are legacy and
   are being folded into the Hydra tree, not extended.
 - **Python is the primary interface; config is a thin wrapper.** Every
   physical parameter and its default value lives once, as a Python default on
   the component class. YAML may carry only wiring (`_target_`, required
   inputs, the non-default choices that define a named configuration) — never a
   parameter default, which would immediately drift from the Python one.
 - Canonical/validated configurations get their own named config file with
   comments explaining WHY each setting is what it is, so a production run is
   one command.
 - One-off experiment scripts (personal paths, GPU indices, ad-hoc drivers) do
   not belong in the repo at all.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [climate-analytics-lab/jax-esm](https://github.com/climate-analytics-lab/jax-esm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
