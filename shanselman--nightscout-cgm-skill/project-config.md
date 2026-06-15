---
trigger: always_on
description: Analyze CGM blood glucose data from Nightscout. Use this skill when asked about current glucose levels, blood sugar trends, A1C estimates, time-in-range statistics, glucose variability, or diabetes management insights.
---


# Nightscout CGM Analysis Skill

This skill provides tools for fetching and analyzing Continuous Glucose Monitor (CGM) data from Nightscout.

## ⚠️ Before Making Changes

**Always run tests before and after modifying `cgm.py`:**

```bash
cd <skill-path>
python -m pytest tests/ -q           # Quick check (281+ tests)
python -m pytest tests/ --cov=scripts  # With coverage
```

## Available Commands

Run the `cgm.py` script from this skill's `scripts/` directory:

```bash
python <skill-path>/scripts/cgm.py <command> [options]
```

Where `<skill-path>` is the location where this skill is installed (e.g., `~/.copilot/skills/nightscout-cgm`, `.github/skills/nightscout-cgm`, or `.claude/skills/nightscout-cgm`).

### Commands

| Command | Description |
|---------|-------------|
| `current` | Get the latest glucose reading |
| `analyze [--days N]` | Analyze CGM data (default: 90 days) |
| `report [--days N] [--output PATH] [--open]` | Generate interactive, PDF-printable HTML report with charts |
| `goals [view|set|clear]` | View, configure, or clear local report goals |
| `compare --period1 P1 --period2 P2` | Compare two time periods side-by-side |
| `alerts [--days N]` | Get trend alerts for recurring patterns |
| `refresh [--days N]` | Fetch latest data from Nightscout |
| `auto-refresh [view|set|on|off]` | Configure refresh-on-query sync, no daemon required |
| `patterns [--days N]` | Find interesting patterns (best/worst times, problem areas) |
| `query [options]` | Query with filters (day of week, time range) |
| `day <date> [options]` | View all readings for a specific date |
| `worst [options]` | Find your worst days for glucose control |
| `chart [options]` | Terminal visualizations (heatmap, sparkline, day chart) |
| `pump` | Get current pump status (IOB, COB, predicted glucose) * |
| `treatments [--hours N]` | Get recent treatments (boluses, temp basals, carbs) * |
| `events [--tag TEXT] [--days N]` | Correlate glucose response around existing Nightscout treatments/events * |
| `ages [--count N]` | Get CAGE/SAGE/IAGE from Site/Sensor/Insulin Change events * |
| `profile` | Get pump profile settings (basal rates, ISF, carb ratios) * |

\* **Optional pump/treatment commands require matching Nightscout uploads.** `pump` and `profile` require Loop, OpenAPS, AndroidAPS, or similar devicestatus/profile data. `treatments`, `events`, and `ages` require treatment/event entries. CGM-only users won't see errors—commands return a structured message explaining that the data is not available.

### Report Command

Generate a comprehensive, self-contained HTML report with interactive charts:
- `--days N` - Number of days to include (default: 90)
- `--output PATH` - Custom output path (default: nightscout_report.html)
- `--open` - Open report in browser after generating

**Auto-Sync:** Reports automatically sync from Nightscout if local data is more than 30 minutes old.

**Report Features:**
- Interactive date controls (7d/14d/30d/90d/6mo/1yr/All + custom date pickers)
- All charts recalculate dynamically in browser
- Sticky section navigation for long reports
- Deterministic executive summary with concise status bullets
- Print / Save PDF action with print-friendly styles
- Time-in-Range pie chart
- Modal Day (24-hour profile with percentile bands and target range lines)
- Daily trends, Day of week comparison
- Glucose histogram, Heatmap with hover tooltips
- Weekly summary with week-over-week TIR delta, best day, and context notes
- Key stats: TIR%, GMI (estimated A1C), CV (variability)
- Goal tracking: configurable TIR, CV, GMI, and average glucose targets
- **Insulin Delivery** (if using Loop/OpenAPS): TDD breakdown (bolus/basal), stacked bar chart, carb tracking

### Goals Command

Configure local report goals stored in `config.json`:
- `goals` or `goals view` - Show active goals
- `goals set --tir PCT --cv PCT --gmi PCT --average MGDL` - Set one or more goals
- `goals clear` - Remove custom goals and return to defaults
- Goal directions: TIR is minimum target; CV, GMI, and average glucose are maximum targets

### Auto-Refresh Command

Configure stale-data sync before read-only queries. This does not run a persistent daemon:
- `auto-refresh` or `auto-refresh view` - Show current refresh-on-query settings
- `auto-refresh set --minutes N` - Sync before queries when the newest local reading is older than N minutes
- `auto-refresh on` / `auto-refresh off` - Enable or disable refresh-on-query
- Use `refresh --days N` for explicit manual sync

### Day Command

View detailed readings for a specific date:
- `day <date>` - Date can be 'today', 'yesterday', '2026-01-16', or 'Jan 16'
- `--hour-start H` - Start hour for time window (0-23)
- `--hour-end H` - End hour for time window (0-23)

### Worst Command

Find your worst days ranked by peak glucose:
- `--days N` - Number of days to search (default: 21)
- `--hour-start H` - Start hour for time window (0-23)
- `--hour-end H` - End hour for time window (0-23)
- `--limit N` - Number of worst days to show (default: 5)

### Query Options

The `query` command supports flexible filtering:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shanselman/nightscout-cgm-skill](https://github.com/shanselman/nightscout-cgm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
