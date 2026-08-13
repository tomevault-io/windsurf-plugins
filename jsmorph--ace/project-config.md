---
trigger: always_on
description: The audience is expert.  No tolerance for bullshit, junk
---

# Style Guide

## Core Principles

The audience is expert.  No tolerance for bullshit, junk
language, guessing, fashion, or anything other than the
highest quality technical work.  No selling, pitching, or
persuading.

Edit yourself like a serious author with an old-school editor
at The New Yorker.  Think John McPhee.  Every word should be
meaningful; remove those that aren't.  Simplicity and clarity
are the greatest virtues.

When in doubt about how to draft something, *ask*.

## Development Practice

**No gratuitous comments** in code or elsewhere.

**Errors**: Do not swallow errors.  Logging an error counts
as swallowing.

**Console output**: No colors.  No ellipsis in logging or
messages.

**Dependencies**: Minimize third-party dependencies.  Ask for
approval before introducing any.

**Design decisions**: Do not make critical design decisions on
your own.  Discuss first.

**Problem-solving**: Work on problems directly.  No hacks or
workarounds.  Search for and use **authoritative
documentation** rather than guessing.

**devnotes.md**: Keep an organized journal of development
notes: links to authoritative documentation, rationales for
decisions, background discussion, and small plans with
checkboxes.

**Commit messages**: First line is a headline, first letter
capitalized (unless a symbol), always under 60 characters, no
period.  The body, if necessary, is concise, grammatical, and
precise.  Omit the body if unnecessary.

## Document Structure

Two or three heading levels suffice for most documents.  No
deep hierarchies.

Prefer tables over many parallel subheadings.  Prefer
paragraphs over lists: the audience reads complete paragraphs
and often prefers them.  Use lists for genuinely enumerable
items, not as a substitute for prose.

Use modest inline markup for emphasis.  Use colons, not
hyphens, to introduce explanatory clauses.

Markdown links to local files: use real titles, not
filenames.  When a filename is required, format it with
backticks.

In a typed document like this, two spaces should precede the
start of a new sentence.

## Language

### Throat-clearing and announcements

Cut sentences that announce what follows rather than saying
it.

| Cut | Replace with |
|-----|--------------|
| "Below is a specification..." | "This specification..." |
| "There is also the matter of X." | Start with X directly |
| "A further limitation is cultural." | State the limitation directly |
| "It is not advocacy." | Delete (defensive) |
| "not merely a technique for X; it is a response to Y" | "a technique for X that addresses Y" |
| "is fundamentally about" | "determines" or state directly |

### Filler words

Delete unless they carry genuine meaning: **simply**,
**itself** (exception: emphasizing identity, e.g., "truth
itself"), **underlying**, **actual**, **clearly**,
**entirely**, **merely** (see throat-clearing above),
**given** (as filler).

### Passive voice

Passive voice hides the actor or weakens the sentence.
Prefer active constructions, but don't go to extremes.

| Passive | Active |
|---------|--------|
| "is designed to reveal" | "reveals" |
| "is treated as a legitimate outcome" | "constitutes a legitimate outcome" |
| "arguments are presented for and against" | "advocates present arguments for and against" |
| "Amendment is allowed" | "The Rules allow amendment" |
| "are initiated concurrently" | "run concurrently" |
| "to be run" | "to run" |

### Weak verbs and hedges

| Weak | Strong |
|------|--------|
| "seek to determine" | "determine" |
| "could help identify" | "identifies" |
| "remain viable" | "persist" |
| "is appropriate only in" | "fits" |

### Jargon and academic hand-waving

Replace bureaucratic, academic, or stilted phrasing with
plain language.

| Jargon | Plain |
|--------|-------|
| "operationally mandatory determinations" | "required decisions" |
| "evidentiary fragility" | "whether the evidence supports it" |
| "unavoidable perception effects" | "random variation in how evidence is weighed" |
| "principled reflection of the evidence" | "honest acknowledgment that the evidence is inconclusive" |
| "the degree to which" | "how well" |
| "well suited to" | "applies to" |
| "not well suited for" | "not designed for" |
| "agnostic as to domain" | "domain-agnostic" |
| "provide a framework for determining" | "determine" |
| "defining feature" | cut; just state what it does |
| "raises similar boundaries" | "faces similar limits" |
| "draws on a tradition" | name the source or cut |
| "reflects X's contention that" | "follows X:" or just state the idea |
| "embodies the intuition" | "implements the idea" |
| "rests on commitments" | "assumes" |
| "has roots in" | cut; name-dropping without substance |

Fields don't do things; people do.  "Social epistemology has
documented" becomes "Research shows" or a specific citation.
"Epistemology has long recognized": cut it and state the
point.

Avoid jargon that sounds impressive but says little:
"convergent truth tracking" means "independent confirmation";
"institutionalized epistemic humility" should say what the
institution does; "epistemological commitments" means
"assumptions."

When tempted to cite a philosopher, ask whether the name adds
information or is decoration.  If decoration, cut it.

### Redundancy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsmorph/ace](https://github.com/jsmorph/ace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
