---
trigger: always_on
description: Guidance for AI-assisted sessions working on this repository.
---

# CLAUDE.md

Guidance for AI-assisted sessions working on this repository.

## What this is

rigc compiles a **rig spec** plus a motion spec — and, for a cut with measured art
behind it, a cut manifest — into Spine 4.3 skeleton data and a one-part-per-page
atlas, then round-trips the result through `@esotericsoftware/spine-core` and 34
named assertions before anything is written. Read [README.md](README.md) for the
formats, the CLI and the assertion list; [`src/rig.ts`](src/rig.ts) is the rig
spec's own documentation.

[docs/AUTHORING.md](docs/AUTHORING.md) is the guide an agent authors *from* — the
two input files field by field, the emission rules, the CLI loop, and the map from
each named failure to the file that has to change. It is a first-class deliverable,
not a summary of this file: the guide and the validator's messages together are the
only interface an agent that cannot see the rig actually has. **Anything that
changes an input format, an error message or an assertion changes that guide too.**

## The doctrine: a tool for AI, not for people

Everything below follows from one observation. An agent authoring a rig cannot see
it. Spine's parser accepts a great deal of wrongness in silence — the constraint
that vanishes, the NaN curve, the mesh that quietly loses its bone weights — so an
agent with only a parser for feedback will report success on a broken rig and be
sincere about it. rigc exists to convert that silence into a named failure.

- **The validator's messages are the UI.** They are what the agent reads and what
  it acts on, so a failure detail must name the object, the value found and the
  value required. `A20_MESH_WEIGHTS_COHERENT: mesh "x" vertex 12 weights sum to
  0.9000` is the product. "invalid mesh" is not.
- **Everything in a rig spec resolves by name, and a miss is refused by name.**
  A bone's `parent`, a slot's `bone`, a constraint's `bones` and `target`, a
  draw-order key's `slot`, and an authored mesh's vertex `weights`. The last of
  those used to be the exception — weights carried raw indices into the *emitted*
  bone array, so inserting a bone rebound every vertex of every mesh below it with
  a green gate and an unmoved `diff` (issue #45, the third example in the
  paragraph above, reproduced rather than caught). Spine's index encoding is still
  reachable behind `"boneIndexing": "raw"`, because what it buys is exactly the
  silence, and that has to be asked for.
- **The compiler never invents a value that is not in the spec.** No defaults
  guessed from the art, no re-measuring of plates, no "reasonable" fallbacks. If a
  number is missing, that is a `CompileError` naming the field. A compiler that
  fills in gaps produces rigs nobody can reason about — and it makes the manifest
  stop being the record of what was measured.
- **Emit only after green.** `build` compiles, validates, and writes *only* if
  every assertion passes. Never reorder that. A wrong file on disk outlives the
  console output that warned about it.
- 🔒 **Validation through spine-core is not optional — this is a structural
  invariant, not a default.** There must never be a `--no-validate` or
  `--emit-anyway` flag, an environment escape, or an exported API that hands back
  emitted artifacts without the round-trip having run. Two reasons, and either one
  alone is sufficient:
  1. **Correctness.** The round trip through the official parser is the only thing
     that makes the output trustworthy; a bypass turns rigc back into a program
     that prints plausible JSON.
  2. **Licensing.** rigc links `spine-core`, so the Spine Runtimes License covers
     running it — see [NOTICE.md](NOTICE.md). A build path that does not link the
     runtime would be a Spine-format emitter with no runtime dependency, i.e.
     exactly the shape of a tool for working around the editor licence. rigc is
     complementary to the Spine editor and must remain structurally incapable of
     being used as a substitute for it. Do not accept a "just for testing" bypass.
- **Determinism is a contract, not a habit.** `A18_DETERMINISTIC_EMIT` compares a
  second, independent compile byte for byte. Anything non-deterministic —
  iteration over an unordered set, a timestamp, a locale-sensitive format, floating
  noise — breaks it, and that is the point.
- **A gate nobody has seen fail is not a gate.** Every assertion needs a mutant in
  `selftest.ts` that makes it fire, and every suite needs a positive control. An
  assertion whose data is absent reports **SKIP**, never a pass — folding vacuous
  checks into the pass count is how a gate comes to look kept while checking
  nothing.
- **No `any`, no `as any`, in `src/` or `cli.ts`.** `selftest.ts` is the one
  exception and it is scoped: the mutants deliberately forge malformed skeleton
  JSON, so they turn the rule off around the mutant tables and back on after.
  Since 2026-08-22 `bun run lint` enforces this rather than a reader, which is
  also what makes the scope of that exemption checkable — the file's
  `eslint-disable` comments now have to actually bracket every `any` in it.

## Conventions

- Bun + TypeScript, ESM, `.ts` extensions in relative imports.
- `src/` is pure: no clock, no randomness, no network. **Two** files link

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firejune/rigc](https://github.com/firejune/rigc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
