---
trigger: always_on
description: <!-- precode:start -->
---

# Writing Standards

<!-- precode:start -->

## Naming

- A name states what the thing is, does, and any side effect: `get_or_create_session()`.
- A name matches its abstraction level: `storage.save()`.
- Context comes from the enclosing class or module: `Address.state`.
- Classes are nouns, methods are verbs, predicates are questions (`is_empty()`); use established CS and domain terms.

## Functions

- A function does one thing at one level of abstraction; extract until that
  is true: `if e.is_payday(): e.deliver_pay(e.calculate_pay())`.
- A signature follows the algorithm's natural statement; lean signatures are
  for orchestration boundaries. Values forming one concept become one object
  (`x, y, radius` → `Circle`); flag-controlled behavior becomes separate
  functions; results come back as return values.
- A repeated type-switch becomes polymorphism; at most one `match` stays,
  in the factory.

## Comments

- Identifiers English; comments, docstrings, commit prose Chinese.
<!-- plain:start -->
- Plain full sentences; no coined shorthand or metaphor terms — a needed
  new term gets a half-sentence gloss at first use.
<!-- plain:end -->
- Present tense, what the code does; intent, rationale, plans never.
- Clean first, then comment.
- Public-API docs follow the surrounding style.
- No commented-out code — git remembers; TODOs get a tracking reference or
  get done.

## The Ladder

Per-tier placement and shape live only in this section; no other section
restates a tier.

| tier | scope | lifespan | anchor | update trigger |
|---|---|---|---|---|
| token micro-comment | one token | with the line | the line below it | edit to the line |
| `//// ////` unit marker | one unit | with the unit | the marker, `rg '////'` | edit to the unit |
| file-head block | one file; a multi-file module via its public-interface file | with the file | H1 module name, `rg '^# '` | edit to a fact it states |
| resident doc | the repo | presumed: every session; admitted: per its named loader | section heading, `rg '^#'` | edit to a rule it states |
| scratch doc | one task | with the task, then archived | date-named file in git-ignored `archive/` | user purge |

Shapes, tier by tier:

- Token micro-comment: `# 加极小值，防分母趋零` above `denom = norm + 1e-4`;
  only where the token's role is invisible yet verifiable from adjacent
  code.
- Unit marker: `//// <做什么> [@user 日期] ////` (after the comment leader)
  above every unit reaching one testable goal. A block unit is wrapped:
  the marker *above* its first line, `//// /<做什么> ////` *below* its last.
- These two are the only comment tiers, nothing between them; each is one
  line directly above its code — the block closer, below, is the sole
  exception.
- File-head block: line 1 the comment leader plus `audience: internal |
  external`; line 2 one Markdown H1, bare module name (`# token-store`);
  the body holds module contract, run prerequisites, and module-wide
  invariants — what holds for the whole file yet no signature states.
- Resident doc: an admitted one opens with the `audience:` tag, the H1
  topic, and one clause of when it applies; it is named by topic in
  kebab-case (`retry-policy.md`), grouped by filename prefix, tree at most
  two levels.
- Scratch doc: any working md (audit, design sketch, one-off report, test
  artifact) under git-ignored `archive/`, named `YYYY-MM-DD-<task>.md` so
  the directory sorts by date; headings allowed — git-ignored, every `rg`
  scan skips it.

Movement:

- A fact lives in the narrowest tier whose scope and lifespan contain it;
  no tier has a line cap, and length never splits a doc.
- Sink: delete a doc line the code already states; a one-file fact sinks to
  that file's head block; a one-unit fact sinks to its marker.
- Float: a fact spanning units floats to the head block; a fact spanning
  modules floats to a resident doc — only through Admission.
- `////` belongs to code units only.

## Format & Structure

- Blank lines separate concepts; related lines stay together.
- A code file stays within a few hundred lines; split it when it outgrows
  that.

## Objects & Data Structures

When writing new code, match the surrounding code's shape; these rules fire
when refactoring.

- A type accumulating getters/setters around exposed state → flatten it to
  plain data: public fields + pure helpers (`v.normalized()`); hiding is
  reserved for boundaries you may swap out.
- The same kind-dispatch repeating → interface + polymorphism; operations
  piling up on the same data → keep the data plain, extract free functions.
- A chain reaching inside a boundary → move the operation behind the
  boundary; chains through plain data stay as they are.

## Boundaries & Third-Party Code

- Third-party types stop at one wrapper; the rest of the code sees your own
  interface (`texture_loader.load()`, vendor API inside).
- Integrating an unfamiliar or unstable dependency → write against the
  interface you wish existed, adapt the vendor behind it; a vendor upgrade
  then touches one file.

## Classes & System Design

- A class has one reason to change; fields used by only a subset of methods
  mark the split seam. Splitting stops when each piece has one job — never
  shard for doctrine. Public surface stays minimal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [x380kkm/380kkm-clean-vibing](https://github.com/x380kkm/380kkm-clean-vibing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
