---
trigger: always_on
description: The shipped program is the design. `README.md` carries the usage path. Six pages under `docs/`
---

# Working in this repo

The shipped program is the design. `README.md` carries the usage path. Six pages under `docs/`
carry the rest — `architecture.md`, `reference.md`, `performance.md`, `measurement.md`,
`instruments.md`, `proof-tools.md` — and the next section says what each is for. When the code
contradicts the intention behind it, report the contradiction instead of redesigning silently.

## Writing prose

Each document has one reader and one job, and a sentence that serves another reader's job moves
or goes.

- `README.md` says what to do. A user reads it to shoot, edit and export. No reasons, no history,
  no measurements.
- `CONTRIBUTING.md` says how to contribute: what runs without a sensor, which checks to run, what
  a pull request says.
- `docs/` say how it works now and what it costs, in present tense. `docs/reference.md` and
  `docs/proof-tools.md` are tables of flags, keys, readings, tools and controls.
  `docs/architecture.md` explains the design as it is. `docs/performance.md` carries the numbers
  with their methods. `docs/measurement.md` says how to take a number. `docs/instruments.md` says
  how to write a check.
- `CLAUDE.md` states rules as present-tense imperatives. A rule stands without the story behind
  it.

No sentence says what something used to be, what shipped once, or what a session did. A past
mistake earns at most one sentence, in the docs page that owns the surface, and only where
knowing it stops a specific mistake. Say what a thing is. Do not say what it is not, or what it
used to be.

## Working with the person who asked

- **Write plainly.** Short sentences, ordinary words, no term the reader did not use first.
- **Less is more in the interface.** Controls and labels explain themselves. Keep text for live
  state, values, errors, accessibility and consequences the controls cannot show. No taglines,
  helper captions, onboarding copy or modal prose.
- **Ask before the work.** An ambiguous requirement gets a question with concrete options.
- **No slop.** No emojis anywhere, no filler, no recap. Say what changed, what it cost, and what
  you did not do.

## Before you commit

A contribution is proven working code, and code on its own is not
([Simon Willison](https://simonwillison.net/2025/Dec/18/code-proven-to-work/)).

- Drive the real surface end to end: `playwright-cli` for the browser, the proof tool for the
  thing it proves. Watch the change happen.
- Write the test for the thing you just did by hand, revert the change, watch it go red, then put
  the change back.
- Name the inputs off the happy path — the empty one, the malformed one, the worst one — and
  either handle them or say in one line what they do.
- Run `node tools/syntax-check.mjs`, `npm run test:unit`, and the proof tools covering the
  surface you touched.
- Report which tools ran, which rows, which numbers, and which checks you skipped.

## What not to build

- **One implementation.** No legacy path beside a new one, no flag to switch between them.
- **No backwards migrations and no compatibility shims.** A capture this build cannot read is
  refused at the door with a reason.
- **No new documents.** A lesson goes in the page that already owns its surface: `README.md`,
  `CONTRIBUTING.md`, `SECURITY.md`, the six pages under `docs/`, `third_party/UPSTREAM.md` or
  `presets-builtin/README.md`.
- **No temporary files in the checkout.** Scratch lives in the session scratchpad. `captures/` is
  gitignored and holds captures.

## Measurement

- Measure. "This should be faster" is not evidence.
- Interleave the A/B. Never take a sequential before/after.
- State window length, sample count, warmup discarded and page-cache state with every number.
- When profiling per-segment cost on the grabber, throw away a run that does not sustain ~30.0
  delivered fps; its timings are noise. A throughput experiment measures that drop instead.
- Use an offline harness for correctness and `grabber --profile` on the sensor for cost.
- Size fixtures by frame count, never by duration.

## Writing a check

`docs/instruments.md` carries the case behind each rule.

1. **Enforce the claim, do not assert it.** Ask what a broken implementation would do to still
   pass, and close that with a falsification control: something that must FAIL when the thing
   under test stops doing the work.
2. **Mutation-test the instrument.** Report which mutations ran. Confirm a missed mutation did
   something, and a caught one was caught for the reason claimed.
3. **Count failed assertions, never exit codes**, and read which fired. Zero failed assertions
   with a non-zero exit is a crash or a printed miss to read, not a catch to record.
4. **Place a probe where its answer would differ.** Arms that agree on a quantity cannot measure
   it.
5. **Look for the object every observation skips**, hardest where the skipping is deliberate.

**Close the class, not the instance.** Make the route table be the dispatch and have the check
walk it, so a route added later is asked by existing.

**Re-run the baseline in the conditions the failure happened in.** A contended machine makes a
check fail in ways that read as a finding.

## Proof tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [totally-tim/braindance](https://github.com/totally-tim/braindance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
