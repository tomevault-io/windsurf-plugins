---
trigger: always_on
description: Numeric forensics for a paper's **supplementary source data** (`.xlsx` / legacy `.xls` / `.xlsm` /
---

# AGENTS.md — paperconan

Numeric forensics for a paper's **supplementary source data** (`.xlsx` / legacy `.xls` / `.xlsm` /
`.csv` / `.tsv`, plus tables inside `.pdf` / `.docx`). It runs a battery of numeric detectors and
surfaces the *locations worth a human re-check* — file, sheet, columns, rows, and the exact rule.

Product/usage docs (install, workflows, report reading, CLI): **[README.md](README.md)**. This file
is the operating guide for AI agents and contributors working *in* the repo.

---

## ⚠️ The one hard rule — neutral language, always

paperconan outputs a **statistical signal, not a misconduct verdict.** In *all* output — reports,
code comments, docstrings, commit messages, PR text, variable names — describe findings as
**"统计信号 / 数据不一致 / 待解释异常 / 请作者澄清 / data inconsistency"**. **Never** write
"fraud", "fabrication", "faked", "misconduct", "guilty", or any accusation of a person. Final
judgement always requires the original data, figure legends, Methods, the authors' response, and
journal/institution review. This red line is non-negotiable and applies everywhere.

---

## Setup

Python ≥ 3.10. The repo uses **uv** (`.venv/`, `uv.lock`, `.python-version`).

```bash
uv sync                       # create/refresh .venv from uv.lock
# or, plain pip for a dev install:
pip install -e ".[dev]"       # engine + pdf/docx extractors + test deps
```

The Rust reader `python-calamine` is a **base** dependency (not optional): it is the *only* reader
for legacy `.xls` / `.xlsm` / `.xlsb` and the fast path for `.xlsx`. PDF/Word table extraction
(`pdfplumber`, `python-docx`) are optional extras, imported lazily.

## Test

```bash
uv run pytest                 # or: .venv/bin/pytest
uv run pytest tests/test_decimal_tail_gate.py -q     # a single file
```

- Golden fixtures in `tests/golden/` and `tests/fixtures/`.
- **Live-network** tests are skipped unless `PAPERCONAN_LIVE=1` (pytest marker `network`).
- Detector correctness is guarded by golden + brute-force-oracle tests (e.g. GRIM/GRIMMER, FDR,
  decimal-tail gate). If you touch a detector, keep these green and add a fixture for the new case.

### False-positive benches

`tests/test_curve_bench_baseline.py` (short-row) and `tests/test_column_pair_bench_baseline.py`
(column-pair) are not unit tests. Each generates its own data, so the answer is known by
construction; runs the shipped detector over it; and freezes what came back. They exist so a
change to a detector's tolerance is argued against a measurement rather than against a corpus
sample, which is not an instrument: the same change measured over a small slice of the corpus and
over a larger one gave opposite answers about part of itself.

Practicalities: `pytest -k "not bench_baseline"` deselects both while iterating. Neither is marked
for opt-in — a bench skipped by default is green while the detector moves, which is the condition
they were written to end. Each carries a `__main__` block that reprints its frozen tables
paste-ready; regenerate that way rather than hand-editing a table.

Five rules, each of which cost review rounds to learn. Where a claim below has a size, the
recipe for measuring it is given instead of the figure — see the last rule for why:

- **Cost is what the AGENT sees, not what the detector emits.** The narrowing here is
  deliberately in the reading: SKILL.md's "Reading A Scan In Layers" has the agent go
  `overview` -> `drill <n>` -> `drill --kind` -> `explain <id>`, and `overview` shows at most
  `DEFAULT_MAX_LOCATIONS` panels. So measure a change by what that page does: whether the ranked
  location list moves, whether a known true signal is on it and at what rank, how many steps
  reach it. Both halves of a trade have to be quoted at that same layer.
  A count taken by calling a detector directly is not that. To see the size of the gap, take the
  densest paper you have and count three things -- what one detector returns, what survives into
  `scan.json`, what `overview` lists. Also compare per paper, over papers that completed under
  every setting: corpus papers differ enormously in size, so a sum over them is largely a
  statement about the biggest, and one paper timing out under some settings and not others can
  move a total further than the settings do. Sort the per-paper counts and see how much of the
  sum the top one carries.
  Read the stage you want to credit before crediting it. Every draft of this rule so far has
  credited a component that does not do the thing. Severity demotion does not gate the reading
  layer: `raw_severity` is frozen before `_demote_dense_relations` runs, and `_raw_severity_of`
  says in as many words that the rewritten field "must not drive routing". Family interleaving
  bounds one family's share of the page rather than preventing it. And what a probe skips
  depends on which detector it calls -- some apply the profile themselves before returning, and
  some are never routed to a panel at all. Each was reached by reasoning from a name, and each
  cost a review round.
  Two corollaries, both measured rather than argued:
  *Many false positives of ONE recognisable kind cost less than their count.* A change that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zixixr/paperconan](https://github.com/zixixr/paperconan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
