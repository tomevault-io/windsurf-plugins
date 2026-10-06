---
trigger: always_on
description: Working rules for coding agents and contributors in Data Universe (Bittensor
---

# AGENTS.md

Working rules for coding agents and contributors in Data Universe (Bittensor
Subnet 13). Facts below were checked against the code on 2026-10-01. If the
code and this file disagree, the code is right: fix this file in the same PR.

## Commands

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt pytest       # pytest is not in requirements

python -m pytest tests/ -q                   # full suite, ~20 s; tests are unittest-style
python -m pytest tests/vali_utils/ -q        # validator suite, run for any validator change
ruff check .                                 # config in pyproject.toml: py310, line length 88

python neurons/miner.py     --netuid 13 --subtensor.network finney --wallet.name W --wallet.hotkey H
python neurons/validator.py --netuid 13 --subtensor.network finney --wallet.name W --wallet.hotkey H
```

Definition of done for a code change: the relevant suite passes, any new
failure is either fixed or shown to be pre-existing on the base branch, and
the PR description states whether the change affects miners, validators, or
both. There is no CI; the suite runs on your machine only.

Known broken, do not "fix" in an unrelated PR: `tests/test_all.py`, and a
handful of tests in `tests/vali_utils/` and `tests/scraping/config/` that fail
on `main` for unrelated reasons.

## Layout

| Path | Owns |
|---|---|
| `neurons/miner.py`, `neurons/validator.py` | Node entry points and background loops |
| `common/` | `DataEntity`, `DataSource`, constants, API client (`api_client.py`) |
| `scraping/` | `Scraper` interface, `ScraperId` registry (`provider.py`), X and Reddit scrapers, coordinator, JSON config |
| `upload_utils/s3_uploader.py` | Miner bulk upload contract |
| `vali_utils/s3_utils.py` | Bulk (S3) validation, `DuckDBSampledValidator` |
| `vali_utils/on_demand/` | On-demand validation, output models, multipliers |
| `vali_utils/miner_evaluator.py` | Per-miner evaluation loop; wires S3 and on-demand results into the scorer |
| `rewards/miner_scorer.py` | Credibility, boosts, coverage multipliers, final score |
| `dynamic_desirability/` | Gravity job list (`total.json`) and validator voting |
| `tests/` | Mirrors the packages above |
| `docs/` | Authoritative docs, listed at the end |

Reports, dumps, logs and parquet files at the repo root are local working
material. Never read them as truth, extend them, or commit them.

## Two reward paths

Every miner is scored on both. They multiply, not add:

```
od  = ondemand_boosts × ondemand_credibility × od_coverage_mult
s3  = min(s3_boost, 2 × od) × s3_credibility ^ 2.5      # 0 when od <= 0
score = min(p2p_score, s3 + od) + s3 + od               # then 70% emission burn
```

A miner with no on-demand activity earns nothing from bulk data. A miner
with bulk data but no live serving is capped at twice its on-demand score.
`rewards/miner_scorer.py:202-247`.

### Bulk data (S3 path)

Miners upload one Parquet snapshot per Gravity job every 2 hours. One
designated validator (`MACROCOSMOS_VALIDATOR_UID`) runs the checks; the
others fetch its published result. A miner is re-validated when its result
is older than 600 blocks.

Upload contract, enforced in `vali_utils/s3_utils.py`:

- Filename `data_YYYYMMDD_HHMMSS_<rows>_<16hex>.parquet`; `<rows>` must equal
  the file's row count. Files under 15 KB or over 1 GB are skipped; any file
  under 50 bytes per row fails the miner outright.
- Snappy or another real codec; an `UNCOMPRESSED` column fails. Row groups
  of exactly 10,000 rows (last one shorter). Every page of every row group
  must decode; a decode error anywhere fails the miner.
- Exact column set per platform (`EXPECTED_COLUMNS_X`, 33 columns;
  `EXPECTED_COLUMNS_REDDIT`, 17). No optional columns. Extra or missing
  columns fail schema. `scraped_at` must be a string column.
- Only the newest file per job counts. Older files in the same job are
  ignored, so incremental uploads score only their last chunk.
- Only jobs present in `dynamic_desirability/total.json` count.

Checks on the sampled files, in order, with the constants that set them:
duplicate normalised URLs within a job (`MAX_DUPLICATE_RATE` 1%), empty
content (`MAX_EMPTY_RATE` 10%), X `view_count` present (`MIN_ENGAGEMENT_RATE`
95%), unique content (`MIN_UNIQUE_CONTENT_RATIO` 10%), transient read
failures (`MAX_TRANSIENT_SKIP_RATE` 30%), job match on label, keyword and
date window (`MIN_JOB_MATCH_RATE` 95%), live re-scrape of 20 sampled
entities (`MIN_SCRAPER_SUCCESS` 80%, overall and per platform with at least
`SCRAPER_PLATFORM_MIN_ENTITIES` 5). Sampling is committed: seeded from a
recent block hash plus the miner's file manifest, so a miner cannot predict
or replay the draw.

Effective size on pass:

```
rows = Σ over active jobs of min(rows_in_latest_file, job.max_rows)   # max_rows default 2,000,000
effective_size = rows × 300 bytes × coverage² × decode_ratio           # coverage = active jobs / expected jobs
s3_boost = effective_size² / Σ all miners' effective_size
```

Credibility: `cred = α + (1−α)·cred` on pass, `cred × (1−α)` on fail,
α = 0.30, start 0.1. Passing never lowers credibility.

### Live queries (on-demand path)

This is the path customers actually pay for and the one most often broken by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [macrocosm-os/data-universe](https://github.com/macrocosm-os/data-universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
