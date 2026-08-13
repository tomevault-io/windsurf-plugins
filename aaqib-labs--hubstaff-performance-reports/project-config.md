---
trigger: always_on
description: Automates bi-weekly workforce performance compliance reports for ~80–90 WebLife Ventures employees tracked via Hubstaff (plus Centrifuse Engineers via TMetric). Reports are published as static HTML via GitHub Pages for internal executive review.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repo Does

Automates bi-weekly workforce performance compliance reports for ~80–90 WebLife Ventures employees tracked via Hubstaff (plus Centrifuse Engineers via TMetric). Reports are published as static HTML via GitHub Pages for internal executive review.

---

## Session Startup Checklist

At the start of every session, before doing anything else:

1. Read this file (`CLAUDE.md`)
2. Read `data/reference/sla_violation_legend.md` — authoritative threshold logic
3. Read `data/personnel/personnel_index.md` — authoritative team/role assignments
4. Check `data/input/` for any new master table CSV files
5. Execute whatever is asked

---

## Folder Structure

```
data/input/
  monthly/           Full calendar month CSVs      → HS-YYYY-MM-master.csv
  biweekly/          Partial/bi-weekly period CSVs → HS-YYYY-MM-DD_to_YYYY-MM-DD.csv
                                                     CE-YYYY-MM-DD_to_YYYY-MM-DD.csv
data/personnel/      Personnel Index — authoritative role/team source
data/reference/      SLA thresholds and violation legend
scripts/             Python report generation scripts
  utils.py           Shared helpers (working-days, proration) — import from here
templates/           Jinja2 HTML report templates
docs/                GitHub Pages source — index.html + all report HTML files
CLAUDE.md            This file
```

**File naming conventions (strictly enforced):**

*CSV inputs (`data/input/`):*
| Type | Format | Example |
|------|--------|---------|
| Hubstaff full month | `HS-YYYY-MM-master.csv` | `HS-2026-03-master.csv` |
| Hubstaff bi-weekly | `HS-YYYY-MM-DD_to_YYYY-MM-DD.csv` | `HS-2026-03-01_to_2026-03-24.csv` |
| Centrifuse Engineers (CE) | `CE-YYYY-MM-DD_to_YYYY-MM-DD.csv` | `CE-2026-05-01_to_2026-05-19.csv` |

*HTML outputs (`docs/`):*
| Type | Format | Example |
|------|--------|---------|
| Hubstaff bi-weekly | `YYYY-MM-DD_to_YYYY-MM-DD_biweekly_top_violators.html` | `2026-03-01_to_2026-03-24_biweekly_top_violators.html` |
| Centrifuse Engineers (CE) | `YYYY-MM-DD_to_YYYY-MM-DD_ce_report.html` | `2026-03-01_to_2026-03-24_ce_report.html` |
| Pattern Analysis (HS) | `YYYY-MM-DD_to_YYYY-MM-DD_pattern_analysis.html` | `2026-01-01_to_2026-03-31_pattern_analysis.html` |
| Pattern Analysis (CE) | `YYYY-MM-DD_to_YYYY-MM-DD_ce_pattern_analysis.html` | `2026-01-01_to_2026-03-31_ce_pattern_analysis.html` |
| Peer Comparison | `YYYY-MM-DD_to_YYYY-MM-DD_peer_comparison.html` | `2026-03-01_to_2026-03-31_peer_comparison.html` |

Note: HTML outputs use date-range prefix + report-type suffix. No `HS-`/`CE-` prefix on outputs — the suffix identifies the source/type. This is by design so `update_index.py` can parse them consistently.

**GitHub Pages serves from `/docs`.** All generated reports are written directly to `/docs/` — there is no separate `/reports/` archive folder. Never rename or remove the `/docs` folder.

---

## Recurring Bi-Weekly Workflow

When asked to "generate the bi-weekly report for [date range]":

1. Find the master table CSV in `data/input/biweekly/`
2. Run `python scripts/generate_biweekly_report.py --input data/input/biweekly/[FILE].csv --start YYYY-MM-DD --end YYYY-MM-DD`
3. Script writes HTML directly to `docs/[start]_to_[end]_biweekly_top_violators.html` and updates `docs/index.html`
4. Commit with message: `Report: Bi-Weekly [start] to [end]`
5. Push to GitHub

If any data anomaly is detected (unexpected columns, missing data, parse errors), flag it before finalising.

---

## Master Table CSV — Column Reference

The input CSV always has these columns (Hubstaff export format):

| Column | Notes |
|--------|-------|
| `Team(s)` | Hubstaff team label — do NOT use for role matching; use personnel_index.md |
| `Member` | Employee name |
| `Activity %` | Keyboard/mouse engagement percentage |
| `Total Worked Hours` | Total logged hours in the period |
| `Break Time` | Raw break hours |
| `Break % of Total` | Break as % of total worked hours |
| `Total Manual Hours` | Manually entered hours |
| `Manual % of Total` | Manual as % of total worked hours |
| `Low Activity Hours (≤20%)` | Hours where activity ≤ 20% |
| `Low Activity % (≤20%)` | Low activity ≤20% as % of total worked hours |
| `Low Activity Hours (≤30%)` | Hours where activity ≤ 30% |
| `Low Activity % (≤30%)` | Low activity ≤30% as % of total worked hours |
| `SLA Violation Legend` | Pre-populated flag string — use as reference only |
| `Red Flag Count` | Pre-populated count — re-evaluate from raw data, do not trust blindly |
| `Yellow Flag Count` | Pre-populated count — re-evaluate from raw data |
| `Total Flags` | Pre-populated count — re-evaluate from raw data |

**Always re-evaluate all flags from raw data.** Pre-populated flag columns are for reference; the scripts are the authoritative source of flag logic.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaqib-labs/hubstaff-performance-reports](https://github.com/aaqib-labs/hubstaff-performance-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
