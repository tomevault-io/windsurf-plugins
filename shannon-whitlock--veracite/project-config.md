---
trigger: always_on
description: VeraCite audits BibTeX/biblatex bibliographies to catch hallucinated, mangled, or
---

# CLAUDE.md — the VeraCite contributor charter

VeraCite audits BibTeX/biblatex bibliographies to catch hallucinated, mangled, or
mis-identified citations before publication. `README.md` is for users,
[`CONTRIBUTING.md`](CONTRIBUTING.md) is the mechanics for human contributors
(setup, workflow, tests, code style), and **this file is the *principles* — the
contract for changing the code** that every contributor, human or AI, should
follow. When a change conflicts with a principle here, the principle wins; if the
principle itself is wrong, change *it* first, in its own commit, with the reasoning.

## Trust is the #1 priority

One false positive on a clean entry makes the user distrust *all* output, so **a
false positive is the cardinal sin — prefer silence to a wrong flag**, tightening
a rule even at some cost to recall, never the reverse.

VeraCite is **read-only**: it flags, never edits. **Crossref (by DOI) is the
primary metadata resource**; arXiv/INSPIRE/OpenAlex/Open Library corroborate it.
A `suggested:` edit always conforms the bib *toward* that record, unless the
record is clearly broken.

A **VeraCite-side failure must never masquerade as a citation problem.** A
TRANSIENT error (rate-limit, 5xx, network drop) is the tool's hiccup, not a
defect in the entry — mark it retryable and re-run that phase on resume, rather
than reporting it as UNVERIFIED. A settled 404 ("no such record") is not retried.
A tail of entries silently going UNVERIFIED from one rate-limit is a trust
failure as real as a false positive.

## Every message must suggest an action

- If a finding implies nothing to do, don't emit it — silence is the clean pass.
- One finding per line, carrying a stable `category`, the offending line, and a
  structured `suggested` patch when one exists. When one fix resolves several
  findings, suppress the dependents (`SUPERSEDES`).
- **The per-record NDJSON is the single source of truth** — every output (the
  terminal report and `--json`) is a parse of it through one record builder and
  one renderer, never a recompute from live objects or a second formatting path.
  No aggregate is stored: summaries are re-derived from the records each run, so
  fresh and resumed runs take the identical path. A phase is marked done **only
  when it actually succeeded** — a transient or failed attempt leaves it undone
  so a later pass retries it. Each record stamps a `checksum` of its source text
  so an edited entry is recomputed from scratch on resume.
- **Keep the NDJSON forward-compatible.** Tolerate a report a *future* version
  wrote: read with `.get`, treat unknown fields/records as opaque and preserve
  them verbatim rather than dropping them.

## Severity means a specific kind of action

| level | meaning | examples |
|-------|---------|----------|
| `[ERROR]` | a syntax error or clearly-broken citation — **must fix** | unbalanced braces, missing `=`, dead DOI, duplicate, id resolves to a different paper |
| `[WARN]` | a real data problem hurting accuracy — **investigate** | year/author/title/volume/pages/journal disagree with the record; sources conflict |
| `[note]` | stylistic or a completeness/portability nudge — usually no render effect | casing, dashes, brace-protection, an abbreviated given name, an invalid-for-biblatex field |

## Everything is a rule

No guessing in the verification path — every finding is a comparison against an
authoritative record or a deterministic rule (`--llm` is the sole exception).

- A per-entry check is `@rule`; a whole-file check is `@file_rule`, both in
  [`rules.py`](veracite/rules.py).
- Every finding carries a stable `category` that sets its severity, group, and
  catalog text. Its metadata lives in [`report.py`](veracite/report.py) and
  [`config.py`](veracite/config.py); [`catalog.py`](veracite/catalog.py)
  introspects them to build `--list-rules`, so it cannot drift. Adding or
  changing a category touches all three — `test_catalog.py` catches a miss.

## Writing a rule that doesn't misfire

A rule must be **principled** and **general enough** to catch a broad class,
narrow enough to **never fire on a valid entry**.

- Generalize to the underlying defect, not the one input that surfaced it.
- Prefer structural signals (check digits, datamodel legality, brace/quote
  balance) over free-text heuristics, which are the likeliest to misfire.
  *Example:* a digit glued to a name looks like a stray year — but a digit
  inside TeX markup (`\hspace{0.167em}`) is not name content; fold markup out
  before scanning, don't pattern-match the raw text.
- Fold away legitimate variation before comparing (name particles/suffixes,
  ISO-4 abbreviations, brace/quote wrapping, `--` vs `-`) — each unfolded thing
  is a false positive waiting. *Example:* arXiv's id lookup serves only the
  **latest** version's title, so a faithfully-cited v1 title can look like a
  mismatch; never conclude "this title exists nowhere" from a title-search index
  alone, since it too holds only the latest title.
- Attach `suggested=` only when certain of the target; otherwise report the
  discrepancy without one. A *resolved but divergent* record is still
  uncertain — soften the claim rather than asserting a mismatch, since the link

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shannon-Whitlock/VeraCite](https://github.com/Shannon-Whitlock/VeraCite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
