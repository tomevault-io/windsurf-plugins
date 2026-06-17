---
trigger: always_on
description: Analyzes Rails applications and generates comprehensive upgrade reports with breaking changes, deprecations, and step-by-step migration guides for Rails 7.0 through 8.1.1. Use when upgrading Rails applications, planning multi-hop upgrades, or querying version-specific changes.
---


# Rails Upgrade Assistant Skill v1.0

## Skill Identity
- **Name:** Rails Upgrade Assistant
- **Version:** 1.0
- **Purpose:** Intelligent Rails application upgrades from 7.0 through 8.1.1
- **Based on:** Official Rails CHANGELOGs from GitHub
- **Upgrade Strategy:** Sequential only (no version skipping)

---

## Core Functionality

This skill helps users upgrade Rails applications through a sequential three-step process:

### Step 1: Breaking Changes Detection Script
- Claude generates executable bash script tailored to the specific upgrade
- Script scans user's codebase for breaking changes
- Finds issues with file:line references
- Generates findings report (TXT file)
- Runs in < 30 seconds
- Lists affected files for Neovim integration

### Step 2: User Runs Script & Shares Findings
- User executes the detection script in their project directory
- Script outputs `rails_{version}_upgrade_findings.txt`
- User shares findings report back with Claude

### Step 3: Claude Generates Reports Based on Actual Findings
- **Comprehensive Upgrade Report**: Breaking changes analysis with OLD vs NEW code examples, custom code warnings with ⚠️ flags, step-by-step migration plan, testing checklist and rollback plan
- **app:update Preview Report**: Shows exact configuration file changes (OLD vs NEW), lists new files to be created, impact assessment (HIGH/MEDIUM/LOW), Neovim buffer list ready

**User Benefits:**
- ⏱️ Saves 2-3 hours per upgrade (automated detection)
- 🎯 90%+ accuracy in finding breaking changes
- 📋 Clear file:line references for every issue
- 🚀 Reports based on ACTUAL detected issues, not hypothetical
- 📊 50% faster upgrades overall

---

## Trigger Patterns

Claude should activate this skill when user says:

**Initial Upgrade Requests (Generate Detection Script):**
- "Upgrade my Rails app to [version]"
- "Help me upgrade from Rails [x] to [y]"
- "What breaking changes are in Rails [version]?"
- "Plan my upgrade from [x] to [y]"
- "What Rails version am I using?"
- "Analyze my Rails app for upgrade"
- "Create a detection script for Rails [version]"
- "Generate a breaking changes script"
- "Find breaking changes in my code"

**After Script Execution (Generate Reports):**
- "Here's my findings.txt"
- "I ran the script, here are the results"
- "The detection script found [X] issues"
- "Can you analyze these findings?"
- *User shares/uploads findings.txt file*

**Specific Report Requests (Only After Findings Shared):**
- "Show me the app:update changes"
- "Preview configuration changes for Rails [version]"
- "Generate the upgrade report"
- "Create the comprehensive report"

---

## CRITICAL: Sequential Upgrade Strategy

### ⚠️ Version Skipping is NOT Allowed

Rails upgrades MUST follow this exact sequence:
```
7.0.x → 7.1.x → 7.2.x → 8.0.x → 8.1.x
```

**You CANNOT skip versions.** Examples:
- ❌ 7.0 → 7.2 (skips 7.1)
- ❌ 7.0 → 8.0 (skips 7.1 and 7.2)
- ✅ 7.0 → 7.1 (correct)
- ✅ 7.1 → 7.2 (correct)

If user requests a multi-hop upgrade (e.g., 7.0 → 8.1):
1. Explain the sequential requirement
2. Break it into individual hops
3. Generate separate reports for each hop
4. Recommend completing each hop fully before moving to next

---

## Available Resources

### Core Documentation
- `docs/README.md` - Human-readable overview
- `docs/QUICK-REFERENCE.md` - Command cheat sheet
- `docs/USAGE-GUIDE.md` - Comprehensive how-to

### Version-Specific Guides (Load as needed)
- `version-guides/upgrade-7.0-to-7.1.md` - Rails 7.0 → 7.1
- `version-guides/upgrade-7.1-to-7.2.md` - Rails 7.1 → 7.2
- `version-guides/upgrade-7.2-to-8.0.md` - Rails 7.2 → 8.0
- `version-guides/upgrade-8.0-to-8.1.md` - Rails 8.0 → 8.1

### Workflow Guides (Load when generating deliverables)
- `workflows/upgrade-report-workflow.md` - How to generate upgrade reports
- `workflows/detection-script-workflow.md` - How to generate detection scripts
- `workflows/app-update-preview-workflow.md` - How to generate app:update previews

### Examples (Load when user needs clarification)
- `examples/simple-upgrade.md` - Single-hop upgrade example
- `examples/multi-hop-upgrade.md` - Multi-hop upgrade example
- `examples/detection-script-only.md` - Detection script only request
- `examples/preview-only.md` - Preview only request

### Reference Materials
- `reference/breaking-changes-by-version.md` - Quick lookup
- `reference/multi-hop-strategy.md` - Multi-version planning
- `reference/deprecations-timeline.md` - Deprecation tracking
- `reference/testing-checklist.md` - Comprehensive testing
- `reference/pattern-file-guide.md` - How to use pattern files
- `reference/quality-checklist.md` - Pre-delivery verification
- `reference/troubleshooting.md` - Common issues and solutions

### Detection Script Resources
- `detection-scripts/patterns/rails-72-patterns.yml` - Rails 7.2 patterns
- `detection-scripts/patterns/rails-80-patterns.yml` - Rails 8.0 patterns
- `detection-scripts/patterns/rails-81-patterns.yml` - Rails 8.1 patterns
- `detection-scripts/templates/detection-script-template.sh` - Bash template

### Report Templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maquina-app/rails-upgrade-skill](https://github.com/maquina-app/rails-upgrade-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
