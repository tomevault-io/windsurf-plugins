---
trigger: always_on
description: When reviewing pull requests, follow these rules strictly.
---

# GitHub Copilot Instructions for Azure-Sentinel Repository

# Copilot Pull Request Review Guidelines

When reviewing pull requests, follow these rules strictly.

## IMPORTANT: Load Custom Instructions Based on File Type

**Before reviewing any file, CHECK if it matches these patterns:**

### For Solution Metadata Files (`Solutions/**/SolutionMetadata.json`)
1. **LOAD and USE:** `.github/instructions/solution-metadata.instructions.md`
2. Apply ALL guidelines from that file
3. Check all validation rules, required fields, and review focus areas from that custom instruction file
4. Do NOT use general instructions for these files - only use the solution-metadata custom instructions

### For Solution Data Files (`Solutions/**/Data/Solution_*.json`)
1. **LOAD and USE:** `.github/instructions/solution-data.instructions.md`
2. Apply ALL guidelines from that file
3. Check all validation rules, required fields, and review focus areas from that custom instruction file
4. Do NOT use general instructions for these files - only use the solution-data custom instructions

### For Release Notes Files (`Solutions/**/ReleaseNotes.md`)
1. **LOAD and USE:** `.github/instructions/releasenotes.instructions.md`
2. Apply ALL validation rules from that file (path, table structure, version format/order, DD-MM-YYYY date format)
3. If this is the **only changed file in the PR**, still run full Release Notes validation and add findings
4. Do NOT use generic fallback comments for these files

### For Analytic Rules Files (`Detections/**/*.yaml`, `Detections/**/*.yml`, `Solutions/**/Analytic Rules/*.yaml`, `Solutions/**/Analytic Rules/*.yml`)
1. **LOAD and USE:** `.github/instructions/detections.instructions.md`
2. Apply ALL validation rules from that file
3. Check all field requirements, MITRE ATT&CK mappings, KQL syntax, and formatting guidelines from that custom instruction file
4. Validate GUID format, rule names, descriptions, severity levels, and other field-based validation rules
5. Do NOT use general instructions for these files - only use the detections custom instructions

### For Hunting Queries Files (`Hunting Queries/**/*.yaml`, `Solutions/**/Hunting Queries/*.yaml`)
1. **LOAD and USE:** `.github/instructions/huntingqueries.instructions.md`
2. Apply ALL guidelines from that file
3. Verify all required fields are present and properly formatted
4. Do NOT use general instructions for these files - only use the huntingqueries custom instructions

### For Playbooks Files (`Playbooks/**/*.json`, `Playbooks/**/README.md`, `Solutions/**/Playbooks/**/*.json`, `Solutions/**/Playbooks/**/README.md`)
1. **LOAD and USE:** `.github/instructions/playbooks.instructions.md`
2. Apply ALL guidelines from that file
3. Validate ARM template structure, metadata fields, parameters, and README requirements from that custom instruction file
4. Check all required sections in README.md and ARM template metadata
5. Do NOT use general instructions for these files - only use the playbooks custom instructions

### For Workbooks Files (`Workbooks/*.json`, `Solutions/**/Workbooks/*.json`)
1. **LOAD and USE:** `.github/instructions/workbook.instructions.md`
2. Apply ALL guidelines from that file
3. Validate workbook JSON structure, required fields, items array, and metadata requirements from that custom instruction file
4. Check all required top-level fields and item structure validation
5. Do NOT use general instructions for these files - only use the workbook custom instructions

### For Parser Files (`Parsers/**/*.yaml`, `Parsers/**/*.yml`, `Solutions/**/Parsers/**/*.yaml`, `Solutions/**/Parsers/**/*.yml`)
1. **LOAD and USE:** `.github/instructions/parsers.instructions.md`
2. Apply ALL guidelines from that file
3. Validate parser syntax, KQL accuracy, YAML structure, and all required fields from that custom instruction file
4. Do NOT use general instructions for these files - only use the parsers custom instructions

### For Building, Packaging, or Validating Solutions
When the user asks to **build**, **package**, **validate**, or **run validations** on a solution:
1. **LOAD and USE:** `.github/instructions/packaging.instructions.md`
2. Apply ALL guidelines from that file — it contains the script path, parameters, fuzzy matching behavior, and report display rules
3. **Always use the build script** at `.script/local-validation/build-and-validate.ps1` — do NOT run validators manually or write your own validation logic
4. **BEFORE running the script, tell the user:** "🔄 Running full build & validation suite for {SolutionName}. This typically takes 3-5 minutes — I'll present the complete report when it finishes." This message MUST appear in chat before the script executes so the user knows work is happening.
5. **The script takes 3-10 minutes.** NEVER tell the user to "check the terminal."
6. Present the full report — the text between `[REPORT_START]` and `[REPORT_END]` — in your chat response
7. This applies to ALL solutions, including newly created ones that may not have a Package/ folder yet

**Trigger phrases:** "build solution", "package solution", "validate solution", "run validations", "run CI checks", "check if solution passes", "create V3 package"

---

## Files and folders to ignore

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
