---
trigger: always_on
description: Temper reference: status
---



# Status: Quality Metrics Dashboard

**Goal:** Display accumulated quality metrics, trends, and learning loop suggestions.

## Execution

### Step 0: Initialize .temper Directory (if missing)

```
If .temper/ directory doesn't exist:
  1. Create structure:
     .temper/
     ├── specs/           # Feature specs (intent.md, plan.md, tasks.md)
     ├── reviews/         # Review reports
     ├── index/           # Semantic index (optional)
     │   ├── modules.json
     │   └── api-surface.json
     ├── metrics.json     # Quality metrics
     └── review-memory.json  # Pattern memory
  2. Initialize metrics.json with schema below
  3. Initialize review-memory.json: { "patterns": {} }
  4. Report: "Initialized .temper/ directory for quality tracking"
```

### Step 1: Read Metrics + Build Hotspots

```
1. Read .temper/metrics.json
2. Read .temper/review-memory.json
3. Read .temper/specs/ to find active specs
4. Scan .temper/reviews/*.md to build a file frequency map
   - Count how many times each file appears as a finding location
   - Compute: issues_per_file = count of findings at file / number of reviews touching file
   - Top 5 files by issue density = hotspots
```

If `.temper/metrics.json` doesn't exist: show "No metrics yet. Run /temper:review or /temper:check to start tracking."

### Step 1.5: Detect MCP Tool Availability

```
Detect which MCP tools are available by checking tool capabilities:

1. code-review-graph:
   - Try calling get_impact_radius_tool with a trivial query (e.g., current file)
   - If tool exists and responds: available
   - If tool not found or errors: unavailable
   - Alternatively, check if MCP tools prefixed with the server name are registered

2. semgrep:
   - Try calling security_check or semgrep_scan_with_custom_rule
   - If tool exists and responds: available
   - If tool not found or errors: unavailable

3. Read tools.mode from .claude/temper.config:
   - auto: report availability, note fallback behavior
   - heuristic-only: report as "disabled (heuristic-only mode)"
   - require: report as "required" — warn if unavailable

4. Compute evidence ratio from metrics.json evidence field:
   - proven + heuristic + semantic = total evidence
   - ratio = proven / total * 100 (if total > 0)
```

### Step 2: Display Dashboard

```
┌─────────────────────────────────────────────────────┐
│ Temper Status — {project-name}                       │
│ Period: Last 30 days                                 │
│                                                      │
│ REVIEWS                                              │
│   Total:           {count}                           │
│   Issues found:    {count}                           │
│   Auto-fixed:      {count} ({%})                     │
│   Manual fixes:    {count}                           │
│   Acceptance rate:  {%}                              │
│                                                      │
│ QUALITY TREND                                        │
│   Coverage:     {old}% → {new}% {↑/↓}               │
│   Avg issues/review:  {old} → {new} {↑/↓}           │
│   Blocked commits: {count}                           │
│                                                      │
│ TECHNICAL DEBT                                       │
│   Debt indicators: coverage {%}, lint violations {n} │
│   Trend: {improving/stable/degrading}                │
│                                                      │
│ HOTSPOTS (most defect-dense files)                   │
│   1. {file} — {N} issues across {R} reviews          │
│   2. {file} — {N} issues across {R} reviews          │
│   3. {file} — {N} issues across {R} reviews          │
│   (None yet — run /temper:review to start tracking)  │
│                                                      │
│ TOP PATTERNS CAUGHT                                  │
│   1. {pattern} ({count}x) {→ AUTO-RULE / suggested}  │
│   2. {pattern} ({count}x)                            │
│   3. {pattern} ({count}x)                            │
│                                                      │
│ LEARNING LOOP                                        │
│   "{pattern}" found in {X}/{Y} reviews.              │
│   Want to add an auto-rule for this?                 │
│   ▸ Yes, add as BLOCK rule                           │
│     Yes, add as WARN rule                            │
│     No, keep as advisory                             │
│                                                      │
│ REVIEW MEMORY                                        │
│   Suppressed patterns: {count}                       │
│   Promoted to rules: {count}                         │
│                                                      │
│ STANDARDS COMPLIANCE                                 │
│   {standard name}: {%} compliant                     │
│   Violations: {count} ({description})                │
│                                                      │
│ ACTIVE SPECS                                         │
│   - {spec} ({status}, {X/Y tasks done})              │
│                                                      │
│ VERIFICATION                                         │
│   Live scenarios: {enabled/prompt/disabled}           │
│   Last run: {date} ({X}/{Y} passed)                  │
│   Mutations: {N} caught, {N} missed                  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
