---
trigger: always_on
description: Public, method-only repository about how Taiwan's TFDA marks a drug permit as
---

# tw-new-drug-signals — agent card

Public, method-only repository about how Taiwan's TFDA marks a drug permit as
"new". M2M English; the deliverable prose in `README.md` and `docs/` is zh-TW
because its audience is Taiwanese clinicians, pharmacists and policy researchers.

Bootstrapped 2026-08-05. Not yet published: no GitHub remote, no push.

## What this repo is

Semantics, failure modes, crosswalk, and a verification protocol for three
non-equivalent "new drug" signals:

1. `restraintItemsCode` code `07 新藥監視` on the per-permit detail record.
2. TFDA 新成分新藥核准審查報告摘要 (the only per-case list TFDA itself names
   新成分新藥).
3. Self-computed moiety first-appearance over all permits including revoked.

## What this repo is NOT

| Not this | Owner |
|---|---|
| Acquisition / scrapers / scheduling | a separate **private** acquisition repo — do not name or link it anywhere in this tree; this file ships publicly too |
| NHI reimbursement rules, drug pricing | `nhi-rule-history` (public) |
| Any data mirror (permit table, ledger, the 273-permit report table) | nobody — regenerate from source |
| Clinical judgement, drug ranking, efficacy claims | out of scope, permanently |
| Bulk loaders, batch queries against government hosts | out of scope, permanently |

Boundary rule, one sentence: **removing it stops you getting the bytes → does not
belong here; removing it makes you believe something false about the bytes →
belongs here.** Phrased without naming the private repo, because the public
README has to state the same boundary and cannot point at a 404.

## Deliberate deviations from the private acquisition repo's house rules

That repo's rules are for standalone executable scrapers. This one ships a
reference library plus an offline test suite, so two rules are deliberately
inverted. Document, don't silently drift.

| Their rule | Here | Why |
|---|---|---|
| One file = one scraper, no sibling imports | `tw_nme_titles.py` imports `tw_permit_id.py` | It needs the 61-entry closed 字別 set. The other three modules ARE standalone, and `test_standalone_modules_really_stand_alone` proves it. README states the copy-set explicitly. |
| Read-only HTTP, never POST | `tfda_lic_query.py` POSTs | The per-permit detail endpoint is POST-only. Mitigated: single permit per invocation, no loop, no `--input-file`, 30s default interval. **Any bulk loader over this endpoint is out of scope for every public repo here** — changing that requires an explicit owner decision, not a drive-by. |

## Layout

| path | purpose |
|---|---|
| `README.md` | zh-TW, decision table first; `## English` section at the end (single file — two READMEs drift) |
| `docs/00`–`08` | one concern each; every page opens with what it answers and closes with a positive AND negative control |
| `measurements.json` | the figure register: one entry per number, plus a `deleted` section recording every figure withdrawn and why |
| `reference/*.py` | stdlib-only reference implementations, each runnable as a CLI |
| `reference/measure_fixtures.py` | recomputes the offline-reproducible entries of `measurements.json`; `make measurements` |
| `reference/tests/` | offline pytest; MUST NOT make network calls |
| `fixtures/` | code-table snapshots + labelled samples; every file carries a retrieval date and "not a mirror". Two of five carry an executable refetch command; the other three say what they would need, because this repo ships no bulk fetcher — do not write a pointer that reads like a recipe |
| `Makefile` | `test` (offline), `measurements` (offline), `check-live` (one manual request), `codes` (two requests, 30s apart). Both network targets use `curl -sS --fail`; a target that pipes an unchecked body into a parser reports a rate-limit as a data change |

## Hard rules

1. **No bare numbers — every figure is registered in `measurements.json`.**
   One entry per figure: `measured_at`, `population`, `method`, `volatility`,
   `source`, `reproduce`, `cited_in`, plus any `caveat` that must travel with
   it. Prose cites the `M-NN` id instead of restating provenance. Cannot supply
   those fields → rewrite as qualitative prose or **delete**, and record the
   deletion with its reason in the `deleted` section. Deleting is a valid and
   frequently correct outcome; thirteen figures were deleted or replaced on
   2026-08-05.
   - `reproduce.kind` is one of `fixture` / `repo-internal` (recomputed offline
     by `make measurements`, and asserted by `test_measurements.py`),
     `public-source`, or `private-data`. The last two must state what they need.
   - **A re-run recipe that does not run is worse than none.** It reads as
     verified. Three shipped recipes were unrunnable (an undefined `titles`
     variable, two `<你的清單表>` placeholders assuming columns this repo never
     builds); all three are now either executable or explicit about the data
     they require.
   - **A figure whose denominator is still moving may not be published as a
     ratio.** Report a floor (`≥ N`) and pin the timestamp to the minute.
     `volatility: moving` entries are required to carry a minute-level
     `measured_at`, and a test enforces it.
   - **Query the composite, not the ingredients.** A difference, ratio or index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [copper0722/tw-new-drug-signals](https://github.com/copper0722/tw-new-drug-signals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
