---
trigger: always_on
description: Generate Allure-ready reports from bigpowers YAML metadata. Reads execution-status.yaml, release-plan.yaml, epic capsules, task YAMLs, cycle-times.yaml, and bug registry to produce allure-results/junit-results.xml, categories.json, and executor.json. Use when preparing progress dashboards, integrating with Allure TestOps, or generating CI reports.
---


# Generate Allure Report

Generate Allure TestOps-compatible reports from bigpowers project metadata. Produces JUnit XML for story-level test results, custom categories for filtering, and executor metadata — all in the `allure-results/` directory.

## Quick Start

```bash
bash scripts/generate-allure-report.sh
```

## What It Produces

Three files in `allure-results/`:

| File | Description |
|------|-------------|
| `junit-results.xml` | One `<testcase>` per story with `<properties>` for risk, security, WSJF, tier, wave, and status. Incomplete stories get a `<failure>` element. |
| `categories.json` | Custom Allure categories for filtering by epic, risk level (P0), and security reviews. |
| `executor.json` | Build metadata — name, type, version from release-plan.yaml, build order. |

## Data Sources

See [REFERENCE.md](REFERENCE.md)

## Verify

```bash
test -f allure-results/junit-results.xml && test -f allure-results/categories.json && test -f allure-results/executor.json
```

## Handoff

- next_skill: null (terminal skill — no downstream workflow step)

---

# generate-allure-report — Reference

## Data Sources

The script reads five YAML sources from the project:

| Source | Path | Fields Used |
|--------|------|-------------|
| Execution status | `specs/execution-status.yaml` | `epics`, `stories`, `development_status` |
| Release plan | `specs/release-plan.yaml` | `release.version`, `release.status`, `bugs` summary |
| Epic capsules | `specs/epics/**/epic.yaml` + `-tasks.yaml` | Epic metadata, task pass/fail counts |
| Cycle times | `specs/metrics/cycle-times.yaml` | Story-level `cycle_minutes`, `bcp_per_hour`, `source` |
| Bug registry | `specs/bugs/registry.yaml` | Bug counts by status and severity |

## Script Body

`scripts/generate-allure-report.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
source "$(dirname "${BASH_SOURCE[0]}")/lib/python-env.sh"
ROOT="$(git rev-parse --show-toplevel 2>/dev/null)" || ROOT="$(dirname "${BASH_SOURCE[0]}")/.."

mkdir -p "$ROOT/allure-results"

$PYTHON - "$ROOT" <<'PY'
import json
import sys
import xml.etree.ElementTree as ET
from pathlib import Path

root = Path(sys.argv[1])
out = root / "allure-results"

# 1. Read execution-status.yaml
exec_status_file = root / "specs" / "execution-status.yaml"
release_plan_file = root / "specs" / "release-plan.yaml"
cycle_times_file = root / "specs" / "metrics" / "cycle-times.yaml"
bugs_registry_file = root / "specs" / "bugs" / "registry.yaml"

sys.path.insert(0, str(root / "scripts" / "lib"))
from simple_yaml import parse_simple_yaml

exec_status = parse_simple_yaml(exec_status_file.read_text()) if exec_status_file.exists() else {}
release_plan = parse_simple_yaml(release_plan_file.read_text()) if release_plan_file.exists() else {}
cycle_times = parse_simple_yaml(cycle_times_file.read_text()) if cycle_times_file.exists() else {"stories": []}
bugs_registry = parse_simple_yaml(bugs_registry_file.read_text()) if bugs_registry_file.exists() else {"bugs": []}

# Build cycle-times lookup
ct_lookup = {}
for ct in cycle_times.get("stories", []):
    if isinstance(ct, dict):
        ct_lookup[ct.get("id", "")] = ct

# 2. Build JUnit XML
stories = exec_status.get("stories", {})

# Counts for testsuite attributes
total_stories = len(stories)
incomplete = sum(1 for s in stories.values() if isinstance(s, dict) and s.get("status") != "done")

testsuite = ET.Element("testsuite", {
    "name": "bigpowers-epic-progress",
    "tests": str(total_stories),
    "failures": str(incomplete),
    "errors": "0",
    "skipped": "0",
})

for story_id in sorted(stories.keys()):
    story = stories[story_id]
    if not isinstance(story, dict):
        continue

    epic_id = story.get("epic", "unknown")
    title = story.get("title", story_id)
    bcps = story.get("bcps", 0)
    status = story.get("status", "backlog")
    risk_max = story.get("risk_max", "none")
    security_max = story.get("security_max", "none")

    # Enrich with cycle-times data
    ct_data = ct_lookup.get(story_id, {})
    cycle_minutes = ct_data.get("cycle_minutes", 0)
    bcp_per_hour = ct_data.get("bcp_per_hour", 0)

    # Time: cycle_minutes * 60 for seconds in Allure display
    time_seconds = cycle_minutes * 60.0 if cycle_minutes else 0.0

    testcase = ET.SubElement(testsuite, "testcase", {
        "classname": epic_id,
        "name": f"{story_id}: {title}",
        "time": str(round(time_seconds, 3)),
    })

    props = ET.SubElement(testcase, "properties")
    ET.SubElement(props, "property", {"name": "risk", "value": risk_max})
    ET.SubElement(props, "property", {"name": "security", "value": security_max})
    ET.SubElement(props, "property", {"name": "bcps", "value": str(bcps)})
    ET.SubElement(props, "property", {"name": "status", "value": status})
    ET.SubElement(props, "property", {"name": "bcp_per_hour", "value": str(bcp_per_hour)})
    ET.SubElement(props, "property", {"name": "lead_time_minutes", "value": str(cycle_minutes)})

    if status != "done":

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielvm-git/bigpowers](https://github.com/danielvm-git/bigpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
