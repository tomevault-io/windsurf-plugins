---
trigger: always_on
description: Guidance for AI coding agents working under `docs/` (and the one doc-duplicate site outside it: `README.md`). Scoped here — not in the root `AGENTS.md` — because none of this applies to source/test work; keeping it out of the root file keeps every non-docs session from loading it.
---

# docs/AGENTS.md

Guidance for AI coding agents working under `docs/` (and the one doc-duplicate site outside it: `README.md`). Scoped here — not in the root `AGENTS.md` — because none of this applies to source/test work; keeping it out of the root file keeps every non-docs session from loading it.

<!--
BENCH-XREF MAP — canonical source: [evaluations/results.md](evaluations/results.md) (Default + Tuned tables per benchmark section).
Every other file below COPIES numbers out of that file by hand; nothing is templated/generated. If you change a
cell in results.md, grep this repo for `BENCH-XREF` (every duplicate site carries that token in an inline HTML
comment) and update every listed sibling. If you change a cell somewhere else first, go fix results.md too —
it's the source of truth, not just another copy.

Discover live: grep -rn BENCH-XREF docs/ README.md
(15 xref comments as of 2026-08-12: 5 in results.md, 6 in docs/trackers/*.md, 3 in docs/index.md, 1 in README.md.
 A 16th lived in .github/copilot-instructions.md until 2026-08-12, when that file was deleted and its guidance
 merged into the root AGENTS.md — which deliberately carries NO benchmark table. Don't add one there.)

## Canonical tables ([evaluations/results.md](evaluations/results.md))
- id=mot17-default       (## MOT17 -> === "Default")
- id=sportsmot-default   (## SportsMOT -> === "Default")
- id=soccernet-default   (## SoccerNet-tracking -> === "Default")
- id=dancetrack-default  (## DanceTrack -> === "Default")
- Tuned tables (all 4 benchmarks) have NO external duplicates — only referenced from within results.md itself.

## Duplicate sites, per canonical row

SORT row (mot17/sportsmot/soccernet/dancetrack):
  -> [trackers/sort.md](trackers/sort.md)                      (Dataset|HOTA|IDF1|MOTA table, full row, first 3 benchmarks only)
  -> [index.md](index.md)                                      (Algorithms table, HOTA column only)
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only)

ByteTrack row (mot17/sportsmot/soccernet/dancetrack):
  -> [trackers/bytetrack.md](trackers/bytetrack.md)            (table, full row, first 3 benchmarks only)
  -> [index.md](index.md)                                      (L13 headline sentence: MOT17 HOTA only; Algorithms table, HOTA column)
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only)

OC-SORT row (mot17/sportsmot/soccernet/dancetrack):
  -> [trackers/ocsort.md](trackers/ocsort.md)                  (table, full row, first 3 benchmarks only)
  -> [index.md](index.md)                                      (L13 headline sentence: MOT17 HOTA only; Algorithms table, HOTA column)
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only)

BoT-SORT row (mot17/sportsmot/soccernet/dancetrack):
  -> [trackers/botsort.md](trackers/botsort.md)                (table, full row, first 3 benchmarks only)
  -> [index.md](index.md)                                      (Algorithms table, HOTA column)
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only)
  -> [trackers/mcbyte.md](trackers/mcbyte.md)                  (BoT-SORT baseline row, matching benchmark tab, full row)

C-BIoU row (mot17/sportsmot/soccernet/dancetrack — the only tracker doc with a DanceTrack row):
  -> [trackers/cbiou.md](trackers/cbiou.md)                    (table, full row, all 4 benchmarks)
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only)
  -> docs/index.md does NOT have a C-BIoU row — don't add one when editing, that's existing scope, not a gap.

McByte row (mot17/sportsmot/soccernet/dancetrack):
  -> [trackers/mcbyte.md](trackers/mcbyte.md)                  (McByte row, matching benchmark tab, full row)
  -> [index.md](index.md)                                      (Algorithms table, HOTA column only; FAQ "Which tracker should I use?" answer: "McByte leads every benchmark in our evaluation")
  -> [../README.md](../README.md)                              (Algorithms table, HOTA column only; includes C-BIoU row too)
     — the index.md FAQ claim above is TRUE only while McByte is bolded-best in all 4 Default tables. Re-verify, don't assume.

## Structural asymmetries (intentional — do not "fix" by adding rows)
- docs/trackers/{sort,bytetrack,ocsort,botsort}.md tables cover MOT17/SportsMOT/SoccerNet only, no DanceTrack row.
- docs/trackers/cbiou.md is the only individual-tracker doc with a DanceTrack row.
- docs/index.md Algorithms table has 5 tracker rows (SORT/ByteTrack/OC-SORT/BoT-SORT/McByte), no C-BIoU row.
- README.md Algorithms table has 6 tracker rows (SORT/ByteTrack/OC-SORT/BoT-SORT/C-BIoU/McByte).
- The root AGENTS.md carries NO benchmark table — it links to evaluations/results.md. That is deliberate;
  don't add a table back (it replaced .github/copilot-instructions.md, deleted 2026-08-12, which had one).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roboflow/trackers](https://github.com/roboflow/trackers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
