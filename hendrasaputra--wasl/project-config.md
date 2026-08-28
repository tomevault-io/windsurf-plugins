---
trigger: always_on
description: **New here? Read [HANDOFF.md](HANDOFF.md) first** — state, map, traps, open threads.
---

# CLAUDE.md — working rules for Wasl

## What this is

**New here? Read [HANDOFF.md](HANDOFF.md) first** — state, map, traps, open threads.
This file is the rules.
**Wasl** (وصل, "the link") — a verifiable genealogy (nasab) explorer. Source of truth is two JSONL files; the HTML is
generated output. Everything asserted must be traceable to an Arabic primary text.

## Non-negotiable rules

1. **No uncited assertion.** Every parent edge, name, alias, kunya and date lives in
   `claims.jsonl` with `work`, `ed`, `vol`, `page`, `loc`, `ar` and `en`. A person row in
   `people.jsonl` carries identity only — no relationships, no dates.
2. **Quotes must be verbatim from `corpus/`.** `validate.py` re-reads the cited page in the
   pinned OpenITI text and fails if the Arabic string is not there. Never hand-type Arabic
   from memory; copy it out of the corpus file.
3. **Never resolve a disagreement by deleting one side.** If sources differ, add every
   reading as its own claim and link them with `variant_of`. The UI shows all of them.
4. **Never invent a date.** Every date claim carries `date_basis`:
   `attested` | `attested_relative` | `derived_from_age_at_death` | `generation_estimate` |
   `unknown`.
   If the source gives no year, the field stays empty. A blank is correct; a guess is not.
5. **New source ⇒ new row in `sources.tsv`** with version URI, URL, editor and edition, and a
   re-run of `fetch.sh`. No claim may cite a work absent from `sources.tsv`.
6. **Model proposes, script verifies, human approves.** An LLM may draft rows. `validate.py`
   decides whether they are real. Never let a draft land without a passing validate.
7. **Source files carry `SPDX-License-Identifier: GPL-3.0-or-later`.** Keep the header when
   adding a file; see LICENSING.md for why the data and the corpus are not the same thing.
8. **`validate.py` shares code with `nasab.py`, so it can agree with its own bug.**
   `test_wasl.py` re-derives page boundaries from the raw file with plain string operations and
   confirms the checker rejects fabricated quotes and wrong pages. Run both. This is not
   theoretical: it is how the repeated-page-marker bug in Ibn Sa'd was found.

## Workflow
```
./fetch.sh          # pull pinned corpus into corpus/ (gitignored, checksummed)
python3 validate.py  # MUST pass before every commit - proves every quote against the corpus
python3 test_wasl.py # independent checks: re-derives page boundaries without nasab.py
python3 test_parsers.py # focused parser regression checks
python3 build.py     # regenerate index.html
python3 tools/build_entries.py --write   # re-pin the biographical entries
python3 tools/build_summaries.py --write # check the anchors, write summaries.jsonl
python3 tools/build_bios.py              # bio/*.html - CI-built, never committed
```

`bio/` is gitignored. The biography pages carry a quarter of a million words of
OpenITI's Arabic, so CI builds them from the fetched corpus and deploys from the artifact;
the staging step deletes `corpus/` and fails if it survives.

`./fetch.sh` verifies the committed checksum by default. Only
`./fetch.sh --refresh-checksums` may deliberately replace the pin after a reviewed source update.

## English references: tried, and dropped

Guillaume's translation of Ibn Hisham was pinned and then removed. Keep it removed unless a
better source appears, and do not repeat the attempt on the same scans:

- The archive.org scan is **431 leaves for 813 pages** - two printed pages per image - so page
  boundaries cannot be recovered from it at all. Any page number derived by locating text
  would be a guess wearing a citation's clothes.
- Guillaume's own index of proper names was the way round that, since the page numbers are
  then the book's rather than ours. But the OCR recovers about 40% of it, and the stretch
  under 'ayn is essentially gone - which is where 'Ali, 'A'isha, al-'Abbas, 'Abd al-Rahman
  and 'Abd al-Muttalib all live. 25 of 47 people at best.
- The OCR runs entries together on one line, so "'Abdul-Malik b. Hassan, 416, 499* 'Abdu
  Yalil b. 'Amr, 614-15" silently gave one man the other's pages until the parser was taught
  to stop at a name.
- **The check could not have caught the worst of it.** Pinning the headword `Husayn` returned
  `Husayn, al, b. al-Humam (P), 43` - a poet - and every automated test passed, because
  re-reading the index line proves the line exists, not that it is the right person's. A
  citation apparatus whose errors are invisible to its own checker is worse than none.

The other English translations need no scan and are honest as plain bibliography: Bewley's
*The Women of Madina* is Ibn Sa'd's volume of women (abridged), Moinul Haq and Ghazanfar
cover volumes I-II, and Landau-Tasseron's *al-Tabari* vol. 39 is a biographical dictionary.
Usd al-Ghaba and al-Isti'ab - which carry most of our entries - have no English translation
at all. None of that can be machine-verified, so if it is ever added it goes in as
bibliography, from a copy in hand, and labelled as such.

**A page milestone can have four digits.** `PAGE_RE` read `\d{3}` and al-Isti'ab paginates
1..1969 in one run across its four volumes, so 286 published claims named a page a tenth of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hendrasaputra/wasl](https://github.com/hendrasaputra/wasl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
