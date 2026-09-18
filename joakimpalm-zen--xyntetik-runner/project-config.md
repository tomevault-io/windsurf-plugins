---
trigger: always_on
description: provides; avoid absolute market-wide claims unless the evidence supports them.
---

# Runner Agent Rules

These rules are mandatory for every AI or LLM agent that works in this repository.
They are not preferences, prompts to reinterpret, or optional process notes.

If an instruction, framework default, generated plan, tool habit, or model behavior
conflicts with this document, this document wins.

## The Five Defining Rules

1. Deep modules
2. Tracer bullets
3. Test-driven development
4. Grill me always
5. Keep the README current

No implementation work starts until these rules have been considered for the task at hand.

## 1. Deep Modules

Design the codebase around deep modules: simple, deliberate public interfaces with
implementation complexity hidden behind them.

Required behavior:

- Prefer a small public API over many shallow helpers leaking across the codebase.
- Keep module boundaries intentional: CLI/server behavior in `main.c`/`server.c`,
  inference flow in `engine.c`, model loading/forward in `model.c`, constrained
  output in `jsonmode.c`/`schema.c`, platform details in `compat.c`, and backend
  details in `cuda.c`/`metal.m`.
- Do not expose internals just to make a quick change easier.
- Lock module behavior with tests through public behavior: CLI output, HTTP
  endpoints, committed smoke scripts, or focused test binaries.

Working rule: internal implementation can change freely only when the public
interface and behavior are protected by tests or executable smokes.

## 2. Tracer Bullets

Build in tiny vertical slices that prove the end-to-end path before expanding scope.

Required behavior:

- Start with the smallest useful observable behavior.
- Touch the real layers needed for the behavior instead of building isolated
  horizontal scaffolding first.
- Validate the slice immediately with tests, execution, or both.
- Use what was learned from that slice before adding the next one.

Working rule: one thin working path is better than many unverified partial layers.

## 3. TDD

Use test-driven development for features, fixes, and behavioral changes.

Required behavior:

- Write one failing test or smoke for one observable behavior.
- Implement the minimum code needed to pass that test.
- Repeat one behavior at a time.
- Refactor only while tests are green.
- Test through public interfaces where possible.

Forbidden behavior:

- Do not write all tests first and all implementation afterward.
- Do not mock internal collaborators unless there is a clear boundary reason.
- Do not assert on incidental implementation details when public behavior can be
  checked.
- Do not add speculative features that are not required by the current test.

Working rule: red, green, refactor; one vertical behavior at a time.

### Mutation testing: make the rebuild provable

When proving a test can fail — breaking a source, running the gate, restoring
it — the restore must land in a LATER whole second than the build that
followed the mutation. `make` compares whole-second mtimes, so a restore in
the same second leaves a stale object linked and the "restored" run reports
the mutated binary's result. Observed twice (2026-08-09, 2026-08-10), both
times as a gate that appeared to pass while testing the wrong binary.

Required behavior:

- After restoring a mutated source, `touch` it and confirm the rebuild
  actually happened (a compile line in the output, or a changed binary
  mtime/hash) before trusting the run.
- Never conclude from a mutation run that a gate is sound without that
  confirmation; a silently skipped rebuild produces exactly the reassuring
  result the exercise exists to disprove.

The same trap applies to any A/B that builds two binaries from one tree
(`git stash` / checkout / branch switch, build, copy, restore, build). Observed
a third time on 2026-08-14, where `touch` was already being used and still did
not help: the touch and the preceding build landed in the SAME whole second, so
make saw the binary as current. Both binaries came out byte-identical and the
comparison drawn from them was vacuous.

Required behavior:

- `sleep 1` before the `touch` that precedes a rebuild, so the source is
  strictly newer than the previous build.
- Confirm the two binaries DIFFER before trusting any number from them —
  by hash, or better, by a behavioral probe that the change should alter. A
  hash can differ for reasons unrelated to the change; a behavioral probe
  cannot.

Working rule: a mutation test proves nothing until the rebuild is proven, and
an A/B proves nothing until the two binaries are shown to differ.

### Every gate needs one absolute anchor

A gate proves a system agrees with the instrument measuring it. That is not the
same as proving the system is right, and the difference has cost this project
real work three times:

- An anchor comparing a build against itself proves the harness is
  DETERMINISTIC, not CORRECT. A research run measured a model whose weights had
  been silently mis-loaded and produced internally consistent numbers for two
  full runs before an absolute check caught it.
- A geometric proxy cannot detect it either: the same broken checkpoint ranked
  its own layers almost identically to the corrected one.
- `--caps` reported `unified_memory: true` on a discrete GPU for a week because

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joakimpalm-Zen/xyntetik-runner](https://github.com/Joakimpalm-Zen/xyntetik-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
