---
trigger: always_on
description: `cja_auto_sdr` generates Solution Design Reference (SDR) documentation from Adobe Customer Journey Analytics data views.
---

# AGENTS.md — CJA Auto SDR Tool Contract

`cja_auto_sdr` generates Solution Design Reference (SDR) documentation from Adobe Customer Journey Analytics data views.

---

## Setup

```bash
uv sync
```

### Auth: Environment Variables

| Variable   | Required | Description            |
|------------|----------|------------------------|
| `ORG_ID`   | Yes      | Adobe Organization ID  |
| `CLIENT_ID`| Yes      | OAuth Client ID        |
| `SECRET`   | Yes      | Client Secret          |
| `SCOPES`   | Yes      | OAuth scopes (from Adobe Developer Console) |
| `SANDBOX`  | No       | Sandbox name           |

### Auth: Profile Alternative

```bash
uv run cja_auto_sdr --profile-add <name>   # create interactively
uv run cja_auto_sdr --profile <name> ...   # use profile
export CJA_PROFILE=<name>                  # set default
```

Profiles stored in `~/.cja/orgs/<name>/`. Profile overrides env vars.

---

## Command Reference

### Discovery

```bash
uv run cja_auto_sdr --list-dataviews [--format json|csv] [--output -]
uv run cja_auto_sdr --list-connections [--format json|csv] [--output -]
uv run cja_auto_sdr --list-datasets [--format json|csv] [--output -]
```

Supports `--filter PATTERN`, `--exclude PATTERN`, `--limit N`, `--sort FIELD`.

### Inspection (per data view)

```bash
uv run cja_auto_sdr --describe-dataview <DATA_VIEW_ID_OR_NAME>
uv run cja_auto_sdr --list-metrics <DATA_VIEW_ID_OR_NAME>    [--format json|csv] [--output -]
uv run cja_auto_sdr --list-dimensions <DATA_VIEW_ID_OR_NAME> [--format json|csv] [--output -]
uv run cja_auto_sdr --list-segments <DATA_VIEW_ID_OR_NAME>   [--format json|csv] [--output -]
uv run cja_auto_sdr --list-calculated-metrics <DATA_VIEW_ID_OR_NAME> [--format json|csv] [--output -]
uv run cja_auto_sdr <dv_id> --stats
```

`--list-metrics`, `--list-dimensions`, `--list-segments`, `--list-calculated-metrics` support `--filter`, `--exclude`, `--sort`, `--limit`.

### Generation

| Format value | Output produced                        |
|--------------|----------------------------------------|
| `excel`      | `.xlsx` workbook (default)             |
| `csv`        | CSV file(s)                            |
| `json`       | JSON file                              |
| `html`       | HTML report                            |
| `markdown`   | Markdown file                          |
| `all`        | All file formats + console             |
| `reports`    | Alias: excel + markdown                |
| `data`       | Alias: csv + json                      |
| `ci`         | Alias: json + markdown                 |

```bash
# Generate default Excel SDR
uv run cja_auto_sdr <dv_id>

# JSON SDR artifact
uv run cja_auto_sdr <dv_id> --format json --output-dir /reports

# Write to specific directory
uv run cja_auto_sdr <dv_id> --format excel --output-dir /reports

# Batch: multiple data views
uv run cja_auto_sdr <dv_id1> <dv_id2> --format ci --continue-on-error

# Run summary for observability
uv run cja_auto_sdr <dv_id> --format json --run-summary-json -
```

### Comparison

```bash
# Live diff of two data views
uv run cja_auto_sdr --diff <dv1_id> <dv2_id> [--format json] [--output -]

# Save snapshot to file (convention: place dv_id before flags)
uv run cja_auto_sdr <dv_id> --snapshot <output_file.json>

# Compare data view against snapshot file
uv run cja_auto_sdr <dv_id> --diff-snapshot <snapshot_file.json>

# Compare against most recent snapshot in snapshot-dir
uv run cja_auto_sdr <dv_id> --compare-with-prev

# Compare two snapshot files (no API calls)
uv run cja_auto_sdr --compare-snapshots <file1.json> <file2.json>

# List snapshots
uv run cja_auto_sdr --list-snapshots [<dv_id>]

# Prune snapshots by retention policy
uv run cja_auto_sdr --prune-snapshots --keep-last 20 --keep-since 30d
```

Key flags: `--changes-only`, `--summary`, `--format json --output -`, `--warn-threshold PERCENT`,
`--auto-snapshot`, `--auto-prune`, `--snapshot-dir DIR`, `--keep-last N`, `--keep-since PERIOD`.

### Governance (Org-Wide)

```bash
# Basic org-wide report
uv run cja_auto_sdr --org-report [--format json] [--output -]

# With clustering
uv run cja_auto_sdr --org-report --cluster

# Force similarity matrix
uv run cja_auto_sdr --org-report --force-similarity

# CI/CD governance gates
uv run cja_auto_sdr --org-report --duplicate-threshold 5 --fail-on-threshold
uv run cja_auto_sdr --org-report --isolated-threshold 0.3 --fail-on-threshold

# Adjust stale-lease threshold (default: 3600s)
uv run cja_auto_sdr --org-report --lock-stale-threshold 900

# Trending
uv run cja_auto_sdr --org-report --trending-window 10

# Compare to previous report
uv run cja_auto_sdr --org-report --compare-org-report prev.json
```

### Diagnostics

```bash
# Look up an exit code
uv run cja_auto_sdr --explain-exit-code 2
```

### Validation

```bash
# Validate config and API connectivity (no data view required)
uv run cja_auto_sdr --validate-config

# Dry-run: validate config without generating reports (requires dv_id)
uv run cja_auto_sdr <dv_id> --dry-run

# Machine-readable config status
uv run cja_auto_sdr --config-status --config-json
```

---

## Exit Codes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brian-a-au/cja_auto_sdr](https://github.com/brian-a-au/cja_auto_sdr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
