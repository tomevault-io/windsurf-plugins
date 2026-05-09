---
trigger: always_on
description: - Use simple words in output.
---


# Always use this approach:

- Use simple words in output.
- Do not say anything inaccurate. If you don't know something, say so. You can also use MCP Perplexity to find answers.
- If something can be done with a script, then create a temporary script in ./tmp/ directory and run this script.
- If something is complex, then create a list of subtasks in ./tmp/todo_[some-description].md and add list of subtasks, use markdown checklist to mark what is done
- Remember the use case of this project/repository


## Project Naming Convention
**IMPORTANT**: The project name is always written as **DRUSCAN** (all uppercase).
- Use "DRUSCAN" in all documentation, code comments, and output
- Never use variations like "DruScan", "Druscan", or "druscan"


## Content Quality Rule
**IMPORTANT**: Keep audit reports concise and focused on value:
- Shorten tables - show only relevant data, not everything available
- Avoid producing unnecessary content that has no value for the reader
- Focus on actionable insights and findings
- Summarize large datasets instead of displaying complete outputs
- Only include details that support decision-making or reveal important issues


## Drupal Version Compatibility
**IMPORTANT**: All audit scripts are designed for **Drupal 8, 9, 10, 11** only.
- Scripts use `watchdog` table (standard in Drupal 8+)
- Commands use Drush 11+ syntax
- Database schema is based on Drupal 8+ structure
- **Drupal 7 is NOT supported** by these audit scripts


## Use case
- **Primary:** Hand over system from one company/agency to another WITHOUT exposing sensitive data, proprietary code, or database content (safe sharing with 3-5 agencies for accurate quotes)
- Check if developer/agency is doing a good job
- Check if system is secure and could be improved
- Estimate maintenance costs and technical debt
- Plan upgrade path to newer Drupal version
- Assess performance bottlenecks and scalability issues
- Verify compliance with accessibility and data protection standards


## Date Verification Rule
**CRITICAL**: Before writing ANY date to .md or .mdc files, ALWAYS check the current system date first using `date` command. Never use example dates or assume the current date.



## Audit Automation Architecture

The audit system uses **two main files**:
1. **`audit.sh`** - Main orchestration script
2. **`commands_registry.sh`** - Central registry of all audit commands

### How It Works

**`audit.sh`** performs:
- Creates timestamped output directory: `audits_reports/{timestamp}_{site_name}/`
- Copies HTML template to output directory
- Detects Drupal document root (web/ or docroot/)
- Loads command definitions from `commands_registry.sh`
- Executes commands and saves results to JSON files
- Validates JSON output and adds metadata

**`commands_registry.sh`** contains:
- Array of command definitions in format: `"section_name|key|type|command"`
- All audit commands organized by section

### Usage

```bash
# Run all sections
./audit.sh site_name production_url

# Run single section only
./audit.sh site_name production_url section_name

# Examples
./audit.sh droptica.com https://droptica.com
./audit.sh droptica.com https://droptica.com code_quality_tools

# Results saved to: audits_reports/{timestamp}_site_name/
```

**Parameters:**
- `site_name` (required) - Name of symlink in `drupal_sites/` directory
- `production_url` (required) - Production URL for testing (used by `performance` and `accessibility` sections)
- `section_name` (optional) - Specific section to audit (e.g., `drupal_modules`, `code_quality_tools`)


## Adding Commands to Registry

### Command Format

```bash
"section_name|key|type|command"
```

**Parameters:**
- `section_name` - Output file name (without extension)
- `key` - Unique identifier for this data point
- `type` - Either `json` (parsed as JSON) or `text` (stored as string)
- `command` - Shell command to execute (must use `ddev` prefix)

### Examples

```bash
# Simple text output
"system_information|php_version|text|ddev exec php -v"

# JSON output
"system_information|drush_status|json|ddev drush status --format=json"

# Using jq with DOCROOT variable
"drupal_modules|custom_count|text|find \${DOCROOT}/modules/custom -name '*.info.yml' 2>/dev/null | wc -l | tr -d ' '"

# SQL query with escaping
"entity_structure|content_types|json|ddev drush sql-query \"SELECT type, COUNT(*) FROM node_field_data GROUP BY type\" --format=json"

# Fallback for optional features
"hacked_check|status|text|ddev drush hacked:list-projects 2>/dev/null || echo 'Hacked module not installed'"
```

### Common Mistakes

```bash
# ❌ WRONG - Missing quotes
drupal_modules|enabled|json|ddev drush pm:list --format=json

# ❌ WRONG - Not escaping inner quotes
"entity_structure|types|json|ddev drush sql-query "SELECT * FROM node""

# ❌ WRONG - Hardcoded path (use DOCROOT variable)
"custom_modules|list|text|ls -1 web/modules/custom/"

# ❌ WRONG - Not using ddev prefix
"system_info|version|text|drush status"
```

### Escaping Rules

1. **Outer quotes**: Always use double quotes `"` for array entry
2. **Inner quotes**: Escape with backslash `\"` for SQL queries
3. **Variables**: Use `\${VARNAME}` for DOCROOT and BASE_DIR
4. **Pipes and redirects**: No escaping needed (`|`, `>`, `2>/dev/null`)

### Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [droptica/druscan](https://github.com/droptica/druscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
