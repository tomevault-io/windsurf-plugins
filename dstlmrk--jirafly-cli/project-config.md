---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jirafly is a CLI tool for Jira analytics and team sprint planning. It calculates metrics that Jira doesn't provide natively, including team capacity planning, work ratio analysis (maintenance vs product development), and sprint statistics.

**Technology Stack:**
- Python 3.12
- Typer CLI framework
- Jira Python library + direct REST API calls
- uv for dependency management
- Pydantic for configuration validation
- PrettyTable for terminal output

## Development Commands

**Setup:**
```bash
# Install dependencies and sync
uv sync --dev

# Install pre-commit hooks
uv run pre-commit install
```

**Running the CLI:**
```bash
# Sprint planning (requires sprint identifier like "6.12")
uv run jirafly planning <sprint_id> configs/team.yaml

# Work ratio analysis
uv run jirafly ratio configs/team.yaml

# Override team member settings via CLI
uv run jirafly planning <sprint_id> configs/team.yaml --member peter=7.0,0.3 --member jane=5.0,0.5
```

**Code Quality:**
```bash
# Format and lint with ruff
uv run pre-commit run --all-files
```

**Adding Dependencies:**
```bash
# Runtime dependencies
uv add $PACKAGE_NAME

# Development dependencies
uv add --dev $PACKAGE_NAME
```

## Configuration

**Environment Variables (.env file required):**
- `JIRA_URL` - Jira server URL (e.g., https://yourcompany.atlassian.net)
- `JIRA_EMAIL` - Email for Jira authentication
- `JIRA_TOKEN` - Jira API token
- `PLANNING_FILTER_ID` - Jira filter ID for planning command
- `RATIO_FILTER_ID` - Jira filter ID for ratio analysis command

**Team Configuration (configs/team.yaml):**
```yaml
members:
  - name: "John Doe"      # Must match Jira assignee display name
    nickname: john        # Short identifier for CLI overrides
    wd: 5.0              # Working days per sprint (0-10)
    vel: 0.4             # Velocity factor (productivity multiplier)

working_days_per_sprint:
  "6.12":
    total: 25.0          # Total team working days for sprint 6.12
```

Team capacity is calculated as: `wd × vel` per member.

## Architecture

**Core Components:**

1. **jira_service.py** - Jira API client
   - Uses Jira Python library for authentication and filter retrieval
   - Uses direct REST API (`/rest/api/3/search/jql`) for pagination (fetches all issues with nextPageToken)
   - Excludes Epic issue types from task lists

2. **models.py** - Data models and formatting
   - `Task` class with `from_raw_issue()` factory method to parse Jira API responses
   - Custom field mappings (HLE, WSJF, Tech Leads, Sprint)
   - Task categorization: Product, Bug, Maintenance, Excluded (based on labels and issue type)
   - Version extraction logic: strips dates and normalizes to X.Y format (e.g., "6.12.0 (16. 9. - 29. 9)" → "6.12")

3. **team_config.py** - Team configuration management
   - Pydantic models for validation: `TeamConfig`, `TeamMember`, `SprintWorkingDays`
   - CLI override system using nicknames
   - Converts between dict formats for internal use

4. **cli.py** - Command implementations
   - `planning` command: Sprint capacity planning with task assignment analysis
   - `ratio` command: Maintenance vs Product work ratio analysis with time tracking
   - Both commands support team member overrides via `--member` flag

5. **utils.py** - Display utilities (PrettyTable formatting, color coding)

6. **sprints.py** - Sprint calendar
   - Sprints are 14 days, Tuesday to the second Monday
   - Everything is derived from one anchor (`6.33` = 2026-07-21), no date table
   - `parse()` accepts `6.34`, `current`, `prev`, `next`, `-2`
   - Only computes within `ANCHOR_MAJOR`; a different major raises `SprintError`

7. **absence_store.py** - Attendance snapshots
   - `build_snapshot()` turns raw absence days into per-member working days
   - Stored in `~/.jirafly/absences.json`, keyed by sprint label; kept outside
     the repo because it holds colleagues' absence data
   - Read by `planning` so it never needs Pinya or a browser

8. **pinya_service.py** - Pinya HR client (absences)
   - No API access, so it calls the same JSON endpoints as Pinya's web UI:
     `POST /Team/GetJson`, `POST /HR/Absence/EmployeeGrid_Read`,
     `POST /Calendar/Team` (Kendo grids, form-encoded, no antiforgery token)
   - Playwright carries the session (Azure SSO + MFA cannot be automated);
     profile in `~/.jirafly/pinya-profile`, headless unless login is needed
   - Optional dependency: `uv sync --extra pinya`

**Key Custom Fields (from models.py):**
- `customfield_11605` - HLE (estimate)
- `customfield_11737` - WSJF (priority score)
- `customfield_11606` - Tech Lead 1st
- `customfield_11634` - Tech Lead 2nd
- `customfield_10000` - Sprint

**Task Categorization Logic:**
- Excluded: Has "RatioExcluded" or "Bughunting" label
- Maintenance: Has "Maintenance" or "DevOps" label
- Bug: Issue type is "Bug"
- Product: Everything else

## Data Flow

**Planning Command:**
1. Load team config (YAML) and apply CLI overrides
2. Fetch tasks from Jira filter (via `PLANNING_FILTER_ID`)
3. Group tasks by assignee and calculate total HLE per member
4. Compare against team capacity (wd × vel)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dstlmrk/jirafly-cli](https://github.com/dstlmrk/jirafly-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
