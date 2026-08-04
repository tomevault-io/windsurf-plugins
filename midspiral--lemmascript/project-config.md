---
trigger: always_on
description: Guidance for AI coding agents working on LemmaScript itself or on projects that use it. Human-oriented docs live in [README.md](README.md), [SPEC.md](SPEC.md), [SPEC_DAFNY.md](SPEC_DAFNY.md), [SPEC_LEAN.md](SPEC_LEAN.md), [DESIGN.md](DESIGN.md), [TOOLS.md](TOOLS.md). This file collects the things that are easy to get wrong if you only read those.
---

# AGENTS.md

Guidance for AI coding agents working on LemmaScript itself or on projects that use it. Human-oriented docs live in [README.md](README.md), [SPEC.md](SPEC.md), [SPEC_DAFNY.md](SPEC_DAFNY.md), [SPEC_LEAN.md](SPEC_LEAN.md), [DESIGN.md](DESIGN.md), [TOOLS.md](TOOLS.md). This file collects the things that are easy to get wrong if you only read those.

## What LemmaScript is

A verification toolchain for TypeScript. The user writes ordinary TS with `//@ ` annotations. `lsc` generates either:
- **Dafny** — one `.dfy.gen` (always regeneratable) + one `.dfy` (the source of truth where proof additions accumulate). Diff must be additions-only.
- **Lean 4 / Velvet / Loom** — four files: `.types.lean` + `.def.lean` are generated; `.spec.lean` + `.proof.lean` are hand-written.

Whatever you do, the TS file is the source of truth for *the program*. The hand-written verification files are the source of truth for *the proof*. Don't conflate them.

## Toolchain commands

```sh
npx lsc gen   --backend=<dafny|lean> path/to/foo.ts   # generate artifacts
npx lsc check --backend=<dafny|lean> path/to/foo.ts   # gen + verify
npx lsc regen --backend=dafny        path/to/foo.ts   # three-way merge after TS changes
```

Default backend is Dafny. Pass `--backend=...` explicitly anyway — case-study CIs and helper scripts all do, and the default has been flipped before.

After editing `lsc` itself (anything under `tools/`), run `npm run build` before re-invoking `npx lsc` — the CLI runs the compiled `tools/dist/lsc.js`, not the TS source.

## File-edit boundaries

**Never edit these — they are regenerated from `.ts`:**
- `foo.dfy.gen` (Dafny)
- `foo.types.lean`, `foo.def.lean` (Lean)

If a generated file is wrong, the fix is in `foo.ts` (or in the emitter under `tools/`), not in the generated file. The `.gen` extension on the Dafny file is a deliberate signal — it keeps Dafny tooling from auto-verifying it and reminds you it is not the source of truth.

**Edit these freely — they hold proofs:**
- `foo.dfy` (Dafny) — copy of `.dfy.gen` plus additions
- `foo.spec.lean` (Lean) — ghost definitions and helper lemmas
- `foo.proof.lean` (Lean) — `prove_correct` plus tactics

For Dafny, the diff between `foo.dfy.gen` and `foo.dfy` must be **additions-only**. You may insert helper lemmas, ghost predicates, asserts, and loop invariants, but you may not modify or delete generated lines. `lsc check` enforces this. (A subtle trap: don't append a trailing comment to a generated line like `{` or `decreases fuel` — that counts as a *modified* generated line. Put your comment on its own new line.)

### Function-level `ensures` for caller composition

A TS `//@ ensures` always emits a separate `<fn>_ensures` **lemma**; the generated Dafny **function** carries only `requires`/`decreases`. Since a Dafny *function* can't invoke a lemma, a *function* caller can't see a callee's postcondition — e.g. `g()` calling `h(f(x))` where `h` requires `arg >= 0` and `f`'s `>= 0` lives only in `f_ensures` will not verify.

Fix: **hand-add the `ensures` to the generated Dafny function** as an addition. Dafny discharges it inline against the body — even recursively (the function's own postcondition serves as the IH, e.g. `ensures scan(...) >= startDay` on a recursive scan). Callers then compose it with no lemma call. Keep it additions-only: put the `ensures` (and any comment) on their own new lines, above the `{`.

## Regen, don't rm

When the TS changes and the existing `foo.dfy` needs to absorb the new generated code:

```sh
npx lsc regen --backend=dafny foo.ts
```

Do **not** `rm foo.dfy foo.dfy.gen && npx lsc gen ...` — that drops every proof addition you (or the previous agent) made in `foo.dfy`. `regen` does a three-way merge against the old `.dfy.gen` and preserves additions. On conflict it restores the original `foo.dfy` and writes the merged result to `foo.dfy.merged` for manual inspection.

### When regen duplicates declarations: delete `foo.dfy.base`

`regen` anchors its three-way merge on `foo.dfy.base` if that file exists, otherwise on the previous `.dfy.gen`. It writes `foo.dfy.base` when it starts a merge and **deletes it only on success** (after verification passes). So if a `regen` ends in `FAILED` (verification) or `CONFLICT`, a **stale `foo.dfy.base` is left on disk** — seeded from that run's old gen.

The next `regen` then anchors on that stale base instead of the current `.dfy.gen`. If the TS changed again in the meantime, the merge base no longer matches either side and `git merge-file` mis-resolves by **appending fresh copies of the changed declaration and everything after it** — the symptom is a cascade of `Error: Duplicate member name: ...` from Dafny.

**Fix:** delete the stale base and regen again:

```sh
rm -f foo.dfy.base
npx lsc regen --backend=dafny foo.ts
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midspiral/LemmaScript](https://github.com/midspiral/LemmaScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
