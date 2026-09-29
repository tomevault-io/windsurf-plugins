---
trigger: always_on
description: A change is not done until the documents that describe it say so. Update them
---

# Working in this repository

## Every change keeps the documentation true

A change is not done until the documents that describe it say so. Update them
in the same pull request as the code, not in a follow-up:

- **User documentation** — `README.md` and `apps/docs` when behaviour, an API,
  a package or a command changes.
- **`CHANGELOG.md`** — a line under *Unreleased* for anything a user or
  integrator would notice, with the reason attached, not only the what.
- **`MIGRATIONS.md`** — when the spec version moves or stored data needs
  converting.
- **Decision records** (`docs/decisions/`) — a new record for every decision
  someone could helpfully undo, in the format `docs/decisions/README.md`
  describes: next free number, the cost paragraph, the alternatives, and a
  **Verified by** line naming the test or gate that fails if it is violated.
  Add it to the index in that README. A decision that changes an old one marks
  the old record `superseded by NNNN` or `reversed` rather than editing it
  away.
- **Architecture** (`docs/architecture/`, arc42) — the section the change
  touches: building blocks for a new package or module, the runtime view for a
  new flow, cross-cutting concepts, quality requirements, verification, and
  risks and debt for anything accepted rather than fixed.
- **Regulatory** (`docs/regulatory/`) — whenever a change affects what the
  software does wrong, how it is verified or what it depends on:
  - `SAFETY-ANALYSIS.md` for a new failure mode, or a new constraint or test
    against an existing one.
  - `SOUP-DECLARATION.md` for a new or changed runtime dependency, required
    environment, or known anomaly. The composition table is checked against the
    manifests by `apps/docs/src/soup.test.ts`, so a new dependency fails a test
    until the table names it at the range the manifest asks for.
  - `LIFECYCLE.md` when the way work is done or verified changes (a new CI
    gate, a new kind of test).
  - `MDR-CONTEXT.md` only if the scope of what this set claims changes.

The regulatory set is not decoration and not marketing. It exists for a
manufacturer incorporating formancy under IEC 62304, who builds their own
assessment on what it says. **A wrong statement there is worse than an absent
one**: an absent one prompts the question, a wrong one answers it incorrectly.
That cuts both ways — a document still listing something as unimplemented after
it ships is wrong too, because a characterisation describes one version and no
other.

Keep the voice of the existing documents: say what it costs and what it does
not do, and never claim more than a test shows.

## A claim in prose is backed by something that fails

Documentation drifts silently, because prose does not break. So where a document
states a fact about the repository, something fails when it stops being true,
and the document says what.

Three shapes, in order of preference —
[0060](docs/decisions/0060-documentation-is-checked.md) has the argument:

1. **Derive it.** `apps/site/decision-records.ts` counts the directory when the
   site is built, so the landing page's figure cannot be stale.
2. **Check it.** `apps/docs/src/soup.test.ts` parses the SOUP composition table
   and compares it to the manifests; `packages/server/src/dockerfile.test.ts`
   derives the image's COPY list from the server's dependency closure. This is
   the right shape when the prose around the fact is worth writing by hand —
   which, in a regulatory document, it is.
3. **Measure it.** See below.

Counts written into prose go stale: the README once said "Forty-eight decision
records" when there were sixty. **Prefer wording without a number.** Where a
number is the point, derive it at build time rather than typing it; where
neither is possible, date it and say it is re-measured rather than incremented.

**When you write a guard, make it fail first.** Revert the thing it guards,
watch it fail, put it back. A test that has never failed has not been shown to
test anything — the repository's ordinary test-first rule
([`LIFECYCLE.md`](docs/regulatory/LIFECYCLE.md)) — and guards are the tests most
likely to be written green and to stay that way for the wrong reason.

Watching it fail once is necessary and not sufficient. Both of these have happened
here:

- **A guard that passes for the wrong reason.** A documentation check matched the
  prose for a phrase like "not released". Reverting the fix left that phrase
  elsewhere in the paragraph, so it passed — green, and asserting nothing.
  **Derive the fact from the code or the data, never from wording.**
- **A guard that cannot run where it matters.** The documentation link check sat for
  weeks in a script no workflow ran, so it fired on the hosting provider after merge
  instead of on the pull request before it. **A guard that is not a gate is a
  comment.** Equally, a check needing git tags or the network answers differently in
  CI than locally, because `actions/checkout` fetches neither — also not a gate, and
  there the honest move is prose with a date rather than a test that lies.

## Measure before you write a number

Numbers here are measured, not estimated. The proof-of-work challenge was
documented at "around a tenth of a second" for 100,000 hashes; measured, it was

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sharkysan/formancy.ai](https://github.com/sharkysan/formancy.ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
