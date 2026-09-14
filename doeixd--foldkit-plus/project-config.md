---
trigger: always_on
description: - **Commit often.** Prefer small, coherent commits over one large one. Commit as
---

# Agent working agreements

## Commit cadence

- **Commit often.** Prefer small, coherent commits over one large one. Commit as
  soon as a unit of work stands on its own (a module, a config, a test file).
- **After every commit, double check the code.** Re-read the diff that was just
  committed, re-run the relevant checks (`pnpm typecheck`, `pnpm test`,
  `pnpm build`), and fix what the check surfaces in a follow-up commit rather
  than letting it accumulate.

## Reviewing a commit

When re-reading a commit, check each of these deliberately:

- **Logic and correctness.** Does it do what the message claims? Trace the real
  control flow, not the intended one.
- **Edge cases.** Empty, missing, duplicate, already-aborted, out-of-order,
  called-twice, called-after-dispose.
- **Synergy with existing features.** Does it compose with what is already here,
  or does it bolt on a second way to do the same thing?
- **Types and TypeScript DX.** No accidental `any` (especially from
  `Parameters<>` on intersections or circular conditionals). Errors should land
  at the mistake and read clearly. Inference should work at the call site
  without annotation ceremony.
- **Comments.** Explain why, not what. Delete any comment that restates the code.
  Doc comments on public API, none on the obvious.
- **Tests.** See below -- they must be able to fail.
- **Security hardening.** Untrusted input crosses a validation boundary before
  anything else; capability and authorization checks cannot be skipped; failures
  do not leak internals.
- **Performance.** Work done once at definition time rather than per call;
  no accidental O(n) lookups or repeated derivation in a hot path.

Fix what the review finds in a follow-up commit rather than letting it sit.

## De-slop

Review for AI slop and remove it. Concretely:

- **Dead abstraction.** Wrappers that only forward to something else, indirection
  files that re-export one module, options nobody passes, type parameters that
  appear once, `_tag` discriminants never discriminated on.
- **Unused exports.** If nothing imports it and it is not deliberate public API,
  delete it. Do not export "just in case".
- **Comments that restate the code.** `// build the map` above a map build.
  Keep the ones that explain a non-obvious why -- a workaround, a subtle
  ordering, a rejected alternative.
- **Doc-comment padding.** A one-line summary beats three sentences of throat
  clearing. No `@param` that restates the parameter name.
- **Ceremonial defensiveness.** Guards for conditions the types already rule
  out, `?? undefined` on an optional call, try/catch that rethrows unchanged.
- **Copy-paste tests.** Near-identical cases that differ by one literal belong
  in a table, and repeated setup belongs in a helper.
- **Inflated prose.** In docs and commit messages, say the thing once. Cut
  "powerful", "seamless", "robust", "simply", and restated section headers.

Prefer deleting code to adding it. The smallest version that a reader
understands on one pass wins.

## Documentation and README standard

A README is an **onboarding document first and a reference second**. The reader
should understand the package's idea, ownership boundary, and normal path before
meeting the full API surface. Do not make a newcomer reverse-engineer the mental
model from a feature tour.

### Required reading order

Prefer this progression for package READMEs and conceptual guides:

1. **What it is.** One short thesis: the problem the package solves and the core
   mechanism it uses.
2. **When it owns the problem.** Say what kind of state/work belongs here, what
   does not, and which neighboring package owns the adjacent cases.
3. **The mental model.** Show the smallest useful lifecycle, equation, or diagram
   before presenting APIs. For stateful systems, name the authoritative owner and
   how information moves.
4. **A sixty-second path.** One minimal, real example that proves the core idea.
   It should teach one mechanism at a time, not demonstrate every feature.
5. **Interpret the example.** Explain what each important call means, including
   what it deliberately does *not* do. Readers should not have to infer whether
   a call performs I/O, owns state, mutates data, or merely describes a contract.
6. **Build outward.** Introduce common workflows and integrations only after the
   basic loop is clear.
7. **Advanced/reference material.** Kernel APIs, transports, adapters, protocol
   details, compatibility aliases, limits, and unusual extension points belong
   after the application-facing story.

A useful default outline is:

```text
What this package is
When to use it / ownership boundaries
Mental model
Install
60-second example
Core concepts
Common workflows
Integration with neighboring packages
Failure / recovery semantics
Advanced / lower-level API
Limits / when not to use it
Reference / compatibility
```

Do not follow the outline mechanically when a package is simpler, but preserve
its direction: **why and model before machinery**.

### Teach ownership before features

Foldkit Plus packages deliberately avoid duplicate state owners. Documentation
must make that visible.

- Say **who owns the authoritative fact or transition**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doeixd/foldkit-plus](https://github.com/doeixd/foldkit-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
