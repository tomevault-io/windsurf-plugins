---
trigger: always_on
description: F# code generator: parses attributed F# source (via `Fantomas.FCS.Syntax`), runs it through
---

# Myriad

F# code generator: parses attributed F# source (via `Fantomas.FCS.Syntax`), runs it through
plugin generators, emits new `.fs` files as an MSBuild pre-build step. See `README.md` for usage,
`DEVNOTES.md` for the MSBuild rebuild-caching mechanics, and
`.claude/skills/myriad-ast-conventions/SKILL.md` for AST-construction conventions when editing
`src/Myriad.Plugins`/`src/Myriad.Core`.

## Current R&D thread: is there a viable successor architecture, and does it earn its keep?

`experiments/` holds an active, evidence-gated exploration split across two intertwined but distinct
lines, both using the same quartet discipline:

1. **Myriad's own architecture** — whether Myriad's core model (untyped-AST parsing → disk-written
   `.fs` files → separate `dotnet build` re-typechecks them) could be replaced by in-process, typed
   FCS hosting (the way Fable already hosts FCS for transpilation), and whether that would actually
   deliver a real capability win or just be architecture novelty.
2. **General F# type-provider headroom** — a separate line using a second checkout,
   `FSharp.TypeProviders.SDK` (added as an extra working directory), asking what the type-provider
   protocol itself can do that the ecosystem isn't using, independent of Myriad's own configuration.

This is exploratory research, not a committed direction: nothing in `experiments/` has been merged
into `src/` or decided as the project's roadmap.

**Methodology:** `experiments/README.md` — a pre-registered hypothesis → design → results →
adversarial-review discipline (adapted from an ML-training experiment convention), enforced by
`dotnet fsi experiments/check-quartets.fsx`. The point is gating: a hypothesis that fails its own
pre-registration, or a spike whose result doesn't survive its own adversarial review, doesn't get
oversold. Read a quartet's `03-review.md` for the honest verdict, not just `02-results.md`.

**Status as of 2026-07-19 — thirty quartets (Q001–Q030), twenty-eight closed, two planned. Full
digest: `experiments/FINDINGS.md`** — read that first, it synthesizes both lines without requiring all
twenty-five `03-review.md`s as context. One-line summary: **nothing has been merged into `src/` from the
architecture-exploration track, and Myriad's real IDE-invisibility gap is narrowed but still not
solved** — twenty-seven quartets have mapped the space and, for the first time, actually closed part of
it: `Q022` hooked Myriad's own MSBuild codegen target into the design-time-build path and confirmed,
against a literal `fsautocomplete` process over real LSP (not `FSharpChecker`-as-library, a standing
gap this file had flagged since Q006), that it makes a generated member appear with zero `dotnet build`
— but only at project load/reload time, not during live source editing (REVISE; see below). Myriad's-
own-architecture line has eight scoped SHIPs (Q002, Q003, Q010, Q015, Q021, Q024, Q025, Q026) and proof that
overclaiming is easy even inside this repo's own discipline, in at least three distinct ways — not just
claiming more than was shown (Q001 NULL, Q006 REVISE, Q014 REVISE — each looked stronger before
adversarial review, and Q021's own results write-up had a secondary claim struck by review too) but also
claiming a negative more absolutely than the evidence supported (Q022's own results doc concluded no
in-session FSAC reload signal existed at all, until review found one) and reporting a single
unrepresentative worst-case condition as the general conclusion (`Q023`'s scale test always edited the
compilation-order *first* file — the maximum-successor, worst-case edit position — and its executor read
the resulting near-cold cost as "caching is largely absent"; review found cost is actually linear in the
number of files *after* the edit and collapses to a cache hit for a tail edit, REVISE). `Q024` then
generalized `Q023`'s corrected finding across scale (a 20-run position × N sweep, N=10 to 300) and
shipped it, scoped: the per-successor marginal cost shows no detectable drift with N, independently
reproduced, though the review knocked down an over-precise "essentially N-invariant" framing to the
better-supported "no detectable systematic drift" and flagged that this is a cost *model*
(`ParseAndCheckProject` on independent files), not a measured FSAC live-editing session — don't cite it
as "FSAC keystroke cost characterized." The general type-provider line shipped a
provenance-enforcement mechanism (Q008/Q09/Q11, reconstructed and reconfirmed after a real
credibility scare — read `FINDINGS.md`'s "gap in this file's own credibility" section before citing
any Thread 2 SHIP verdict) and then spent five more quartets (Q016–Q020) probing routes around the one
wall that keeps recurring: a type provider can never see a type from the compilation currently in
progress, only already-compiled referenced code (Q006). Every route since has hit its own real, narrower
ceiling: cross-project satellite-DLL forwarding into Myriad's own compiled output works but live
re-exposure has no in-process fix on Windows (Q016–18, settled — use `<ProjectReference>` for that
case); an erased provider that self-parses a source file sidesteps the wall entirely but only ever

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoiraeSoftware/myriad](https://github.com/MoiraeSoftware/myriad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
