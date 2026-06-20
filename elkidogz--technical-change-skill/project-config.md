---
trigger: always_on
description: |
---


# /tc — Technical Change Tracker

Track every code change with structured JSON records and accessible HTML output.
Ensures AI bot sessions can resume seamlessly when previous sessions expire or are abandoned.
Designed for deployment across multiple projects.

## First-Use Detection (MANDATORY — Every Session)

At the start of EVERY session, before doing any work:

1. Check if `docs/TC/tc_config.json` exists in the current working directory
2. **If it EXISTS**: follow the Session Start Protocol in the `/tc resume` section
3. **If it does NOT exist**: prompt the user:
   > TC tracking is not initialized in this project. Would you like to set it up?
   > This enables structured change tracking, AI session handoff, and HTML documentation.
   > Run `/tc init` to get started.
4. Wait for the user's response. If they agree, run `/tc init`.
5. If the user declines, continue without TC tracking for this session.

A global skill is installed at `~/.claude/skills/tc.md` to ensure this check runs
in every project, even those that haven't been initialized yet.

## Overview

Each Technical Change (TC) is a structured record that documents:
- **What** changed (files, code, configuration)
- **Why** it changed (motivation, scope, design decisions)
- **Who** changed it (human or AI bot session)
- **When** it changed (revision history with timestamps)
- **How it was tested** (test cases with evidence from logs)
- **Where work stands** (session handoff data for bot continuity)

### Storage Location
Each project stores TCs at `{project_root}/docs/TC/`:
```
docs/TC/
├── tc_config.json          # Project settings
├── tc_registry.json        # Master index
├── index.html              # Dashboard
├── records/
│   └── TC-001-MM-DD-YY-name/
│       ├── tc_record.json  # System of record
│       └── tc_record.html  # Human-readable
└── evidence/
    └── TC-001/             # Log snippets, screenshots
```

### TC Naming Convention
- **Parent TC**: `TC-NNN-MM-DD-YY-functionality-slug` (e.g., `TC-001-04-03-26-user-authentication`)
- **Sub-TC**: `TC-NNN.A` or `TC-NNN.A.1` (letter = revision, number = sub-revision)
- NNN = sequential number, MM-DD-YY = creation date, slug = kebab-case functionality name

### Implementation States
```
planned → in_progress → implemented → tested → deployed
   │           │              │           │         │
   └→ blocked ←┘              └→ in_progress ←──────┘
   │    │  │                     (rework/hotfix)
   │    │  └→ paused → in_progress
   │    │       │
   │    └──────→└→ voided (terminal — cancelled)
   └→ voided
```

- **paused**: Development work temporarily stopped. Can resume to `in_progress` or cancel to `voided`.
- **voided**: TC cancelled entirely. Terminal state — cannot transition out.

---

## Commands

### /tc init
Initialize TC tracking in the current project. Run this once per project.

**Steps:**
1. Check if `docs/TC/tc_config.json` exists. If yes, report "Already initialized" with current stats and stop.
2. Detect project name: try CLAUDE.md first heading, then package.json name, then pyproject.toml name, then directory basename. Confirm with user.
3. Create directories: `docs/TC/`, `docs/TC/records/`, `docs/TC/evidence/`
4. Create `tc_config.json`:
   ```json
   {
     "project_name": "<detected>",
     "tc_root": "docs/TC",
     "created": "<ISO 8601 now>",
     "skills_library_path": "<absolute path to skills_library/TC>",
     "auto_track": true,
     "auto_regenerate_html": true,
     "auto_regenerate_dashboard": true,
     "default_author": "Claude",
     "categories": ["feature","bugfix","refactor","infrastructure","documentation","hotfix","enhancement"]
   }
   ```
5. Create `tc_registry.json`:
   ```json
   {
     "project_name": "<name>",
     "created": "<ISO 8601>",
     "updated": "<ISO 8601>",
     "next_tc_number": 1,
     "records": [],
     "statistics": {
       "total": 0,
       "by_status": {"planned":0,"in_progress":0,"blocked":0,"implemented":0,"tested":0,"deployed":0,"paused":0,"voided":0},
       "by_scope": {"feature":0,"bugfix":0,"refactor":0,"infrastructure":0,"documentation":0,"hotfix":0,"enhancement":0},
       "by_priority": {"critical":0,"high":0,"medium":0,"low":0}
     }
   }
   ```
6. Generate empty dashboard: run `python "<skills_path>/generators/generate_dashboard.py" "docs/TC/tc_registry.json"`
7. Update CLAUDE.md: read existing file (or create new). Check for marker `## Technical Change (TC) Tracking (MANDATORY)`. If not found, append the contents of `init/claude_md_snippet.md` with `{skills_library_path}` replaced with the actual absolute path.
8. Update `.claude/settings.local.json`: read existing file (or create `{"permissions":{"allow":[]}}`). Merge TC permissions from `init/settings_template.json` (with paths substituted). Deduplicate. Write back.
9. Report all created/updated files. Suggest `/tc create` as next step.

### /tc create <functionality-name>
Create a new TC record.

**Steps:**
1. Read `docs/TC/tc_registry.json`
2. Generate TC ID: `TC-{next_tc_number:03d}-{MM-DD-YY}-{slugify(name)}`
3. Ask user for:
   - Title (default: formatted version of the slug)
   - Scope: feature, bugfix, refactor, infrastructure, documentation, hotfix, enhancement
   - Priority: critical, high, medium, low (default: medium)
   - Summary (at least 10 characters)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elkidogz/technical-change-skill](https://github.com/Elkidogz/technical-change-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
