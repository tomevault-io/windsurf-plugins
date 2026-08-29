---
trigger: always_on
description: Codex is a new programming language, self-sustaining compiler, tools, operating system, repository protocol, trust lattice, encoding, and more. We take the best of type theory, language design, aesthetics, security research, and actual practice. We leave everything else behind. If we didn't build it, we don't trust it. Codex is a new computational substrate intended to be impervious to all currently known attack vectors by-design.
---

# CLAUDE.md -- Codex Project Instructions

## What This Is

Codex is a new programming language, self-sustaining compiler, tools, operating system, repository protocol, trust lattice, encoding, and more. We take the best of type theory, language design, aesthetics, security research, and actual practice. We leave everything else behind. If we didn't build it, we don't trust it. Codex is a new computational substrate intended to be impervious to all currently known attack vectors by-design.

The project was started 3/14/2026.

### The Founding Vision (docs/PM/Stories/Vision/NewRepository.txt)

The original prompt that started the project:

> the new repository. condense all the good ideas humans have had in
> github, sourceforge, etc into a new language. start from first
> principles, find the best implementation, the best abstraction. port it
> to a language that can be transpiled to any old human designed language,
> it abstracts them all into a single perfect language. it is the basis
> for all future code. it exists for human reading and machine. it should
> read like a book. fulfill liskov's hopes for cobol. then we delete
> github and sourceforge entirely fully replaced with a single, ideal
> solution. write the book.

From there the design grew into: a literate-programming language where
prose is load-bearing, a type system with dependent types / linear types /
effect types, a content-addressed repository protocol replacing Git
(facts, proposals, verdicts, trust lattice), a unified environment
(Reader, Writer, Verifier, Explorer, Executor, Narrator, Historian), and
transpilation targets from Rust to WASM to LLVM IR. The full founding
document is in the file above.

## Session Start

**On session start, run `/init`.** This is non-negotiable. The `/init`
skill loads memory, gathers fleet state through parallel agents, reads
the lesson index, and checks Perforce. Do not skip it. Do not
substitute your own init sequence. The skill is at
`.claude/skills/init/SKILL.md`.

If the user's first message asks you to initialize, run `/init`. If
you are unsure whether init has been done, run `/init`.

### The reading model (redesigned 2026-07-28, Damian's direction)

Init used to read ~190k tokens of documents directly into context
before any work started; measured, one session arrived at 59 per cent
spent after one unit of work. Init now keeps in direct context only
what changes behavior at session start (~17k: memory, the lesson index
`docs/PM/Active/Stories/LESSONS.md`, three haiku-agent summaries of
CurrentPlan + Perforce process + active designs, Perforce state).
Everything
else moved to an ON-DEMAND CONTRACT: the skill's Step 5 table maps
each subject to the doc that is mandatory reading BEFORE touching that
subject (`.codex` source -> DevelopersGuide; allocators ->
ArchitectsSketchbook; builds/VM -> OperatorsManual; tests ->
ExaminersAssay via Grep; and so on). The stories in
`docs/PM/Active/Stories/` are no longer read wholesale: LESSONS.md
carries one id per lesson, and **the story behind an id is read in
full the moment that lesson becomes load-bearing for your work** --
that rule is what keeps the summaries-rot failure from coming back.
The reference docs did not move and are not summarized; only WHEN they
are read changed.

## Document Lifecycle

`docs/PM/BACKLOG.md` was deleted 2026-07-23. **Do not recreate it.**
Application-domain registers (`apps/<app>/<app>-backlog.md`,
`codex/<quire>/<quire>-backlog.md`) are unaffected, and an item that
originates in ONE app or quire belongs in that register rather than in
the plan.

**The workplans were emptied and the findings-outbox channel was retired
2026-08-08 at Damian's direction.** `docs/Agents/<agent>-workplan.md`
still exists and **is scratch for the current session's lane state only**:
what is shelved, what is mid-gate, what the next action is. It is emptied
at handoff, not appended to. **Open work does not go in it** -- cross-lane
items go to `docs/PM/CurrentPlan.md`, which is the fleet's only cross-lane
register, and an item originating in one app or quire goes to that
register.

**Do not start a findings outbox anywhere.** A finding worth another
lane's attention goes into the doc that owns the subject the moment it is
verified: the reference docs (`OperatorsManual`, `ExaminersAssay`,
`DevelopersGuide`, `HardwareSitting`), the design that owns the
capability, `LESSONS.md` for a lesson, or the relevant backlog for a gap.

That is the fix for two failures the old arrangement kept producing. A
durable fact parked in a status file is read once at init and then
reasoned about from memory instead of re-read. And an outbox entry was
"deleted by the addressee" from the AUTHOR's file in the author's stream,
which is a cross-workspace write on somebody else's document, so it
almost never happened: a true, unfixed finding about a switch that does

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damiant3/Cobblestone](https://github.com/damiant3/Cobblestone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
