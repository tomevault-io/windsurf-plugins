---
trigger: always_on
description: This file provides context for GitHub Copilot and other AI coding assistants working with this project.
---

# Copilot Instructions for jamf-reports-community

This file provides context for GitHub Copilot and other AI coding assistants working with this project.

## Project Overview

A single-file Python script (`jamf_reports_community.py`, ~3,800 lines) that generates multi-sheet Excel workbooks from Jamf Pro CSV exports and/or jamf-cli JSON data. Configuration-driven: users edit `config.yaml` to map their Jamf Pro column names to logical field names; no Python code changes are needed for normal use.

**Key constraint:** Must work for any organization running Jamf Pro without requiring hardcoded org-specific values (column names, policy names, IP addresses, etc.) in the code.

## Architecture

### Single-File Design

The entire implementation lives in `jamf_reports_community.py`. Do not split into modules—it's designed to be dropped into any directory and run as-is.

### Core Classes

| Class | Purpose |
|-------|---------|
| `Config` | Loads `config.yaml`, deep-merges with `DEFAULT_CONFIG`, exposes typed properties via `resolve_path()` |
| `ColumnMapper` | Resolves logical field names → CSV column names. Key methods: `.get(field)`, `.extract(row, field)` |
| `JamfCLIBridge` | Subprocess wrapper for jamf-cli. Saves JSON to `jamf-cli-data/`. Optional—gracefully no-ops if unavailable. Supports multi-tenant via `profile`. Falls back to cached JSON when live calls fail. |
| `CoreDashboard` | Generates 9 sheets from jamf-cli JSON: Fleet Overview, Inventory Summary, Security Posture, Device Compliance, EA Coverage, EA Definitions, Software Installs, Policy Health, Patch Compliance. |
| `CSVDashboard` | Generates sheets from Jamf Pro CSV export (only when `--csv` provided): Device Inventory, Stale Devices, Security Controls, Security Agents, Compliance, plus one sheet per `custom_eas` entry. |
| `ChartGenerator` | Generates matplotlib PNG charts and embeds in xlsx. Skipped if matplotlib not installed (`HAS_MATPLOTLIB` flag). |

### CLI Commands

```bash
# Generate report from CSV ± jamf-cli data
python3 jamf_reports_community.py generate [--config config.yaml] [--csv path/to/export.csv]
                                           [--out-file report.xlsx]
                                           [--historical-csv-dir snapshots/]

# Fetch live jamf-cli snapshots and archive CSV
python3 jamf_reports_community.py collect [--config config.yaml] [--csv path/to/export.csv]
                                          [--historical-csv-dir snapshots/]

# Export wide inventory CSV from jamf-cli
python3 jamf_reports_community.py inventory-csv [--config config.yaml]
                                                [--out-file inventory.csv]

# Scaffold config.yaml from CSV headers (fuzzy-matches via COLUMN_HINTS/COLUMN_EXCLUDES)
python3 jamf_reports_community.py scaffold [--csv path/to/export.csv] [--out config.yaml]

# Validate jamf-cli auth and config column mappings
python3 jamf_reports_community.py check [--csv path/to/export.csv]
```

## Critical Invariants (Do Not Break)

1. **`_safe_write()` for all CSV-sourced data.** Never call `worksheet.write()` directly with user data. Always route through `_safe_write()` which sanitizes None, NaN/inf, control chars, and formula injection. Static labels (written by script) can use `worksheet.write()` directly.

2. **No hardcoded column names.** All column names come from config via `ColumnMapper`. Strings like `"Computer Name"` appear only in `config.example.yaml` and `config.yaml`, never in code.

3. **No hardcoded org-specific values.** No IP addresses, URLs, usernames, department names, policy names, or EA names in code.

4. **jamf-cli is optional.** Always check `JamfCLIBridge.is_available()` before jamf-cli calls. Script continues with CSV-only output if jamf-cli absent.

5. **matplotlib is optional.** Use `_load_matplotlib()` before chart code. Gate all chart logic on `HAS_MATPLOTLIB`.

6. **Single file—always.** Drop-in script design. No modules, no package structure.

## Config System

### Single Source of Truth

`DEFAULT_CONFIG` (top of script) defines all keys. `config.example.yaml` mirrors that structure exactly—no phantom keys.

**Rule:** Never add a config key to `config.example.yaml` that isn't read by the code.

### When Adding a Config Key

1. Add to `DEFAULT_CONFIG` with sensible default
2. Read it in the relevant class/function
3. Document in `config.example.yaml` with comment
4. Update `README.md` if user-facing

### Key Names (Common Confusion Points)

Use these exact names:

| Section | Key | ❌ NOT |
|---------|-----|--------|
| `columns` | `operating_system` | `os_version` |
| `columns` | `last_checkin` | `last_contact` |
| `columns` | `email` | `assigned_user_email` |
| `jamf_cli` | `profile` | `jamf_profile` |
| `jamf_cli` | `allow_live_overview` | `live_overview` |
| `security_agents` | `connected_value` | `installed_value` |
| `compliance` | `failures_count_column` | `failed_count_column` |

## Testing & Validation

### Manual Test Workflow

No automated test suite. Test CSV: `Jamf Reports/97 Computers.csv` (96 sanitized dummy devices).

```bash
cd /path/to/jamf-reports-community

# 1. Verify syntax
python3 -c "import py_compile; py_compile.compile('jamf_reports_community.py', doraise=True)"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonyyo11/jamf-reports-community](https://github.com/tonyyo11/jamf-reports-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
