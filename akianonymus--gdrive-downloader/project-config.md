---
trigger: always_on
description: - Runs `shfmt` to format all scripts (4 spaces, bash variant)
---

# gdrive-downloader Agent Guidelines

## Build/Lint/Test Commands

### Format and Lint
```bash
./format_and_lint.sh
```
- Runs `shfmt` to format all scripts (4 spaces, bash variant)
- Runs `shellcheck -o all -e SC2312` for linting
- Formats scripts in: `.`, `src/common`

### Merge and Release
```bash
./release.sh
```
Merges and minifies scripts into `release/gdl`

### Run All Tests
```bash
./test.sh
```
Runs integration tests for bash variant with and without --key flag

### Run Single Test
Tests are run via `test.sh` which calls the `_test` function. To run a specific test:
```bash
# Modify test.sh to comment out tests you don't want to run, then:
./test.sh
```

Note: There is no individual test runner - modify test.sh to run specific tests

### CI/CD
The GitHub workflow (.github/workflows/main.yml) runs:
1. Installation tests
2. Format, lint, and release tests
3. Download tests

## Code Style Guidelines

### File Structure
- `src/common/*.sh` - Shared utilities (Bash 4.x+)
- `src/gdl.sh` - Main entry point
- `release/bash/gdl` - Merged bash script

### Formatting (EditorConfig)
- Indent: 4 spaces
- End of line: LF
- Charset: UTF-8
- Trim trailing whitespace: yes
- Insert final newline: yes
- shell_variant: bash
- switch_case_indent: true
- space_redirects: true

### Shebang
```bash
#!/usr/bin/env bash  # All scripts use Bash 4.x+
```

### Shellcheck Directives
```bash
#!/usr/bin/env sh
# shellcheck source=/dev/null        # Top of every sourced script
# shellcheck disable=SC2317          # For unreachable functions
# shellcheck disable=SC2089/SC2090   # For eval usage
```

### Function Naming
All functions use underscore prefix:
```bash
_function_name() { ... }
```
Examples: `_download_file`, `_check_id`, `_api_request`, `_bytes_to_human`

### Variable Naming
Local variables within functions use `_` prefix with function name for scoping:
```bash
_function_name() {
    local_var_function_name="${1:?}"
    another_var="${2:-default}"
}
```

Global exports:
```bash
export API_URL="https://www.googleapis.com"
export DOWNLOADER="curl"
```

### Function Documentation
Standard format:
```bash
###################################################
# Brief description of function purpose
# Required Arguments: 1
#   ${1} = arg1 description
#   ${2} = arg2 description (optional)
# Result: What the function does/returns
# Reference:
#   https://url-to-reference (optional)
###################################################
_function_name() {
    ...
}
```

### Error Handling
- Use `|| return 1` pattern: `command || return 1`
- Mandatory arguments: `var="${1:?Error: Missing argument}"`
- Early return: `[ -z "${var}" ] && return 1`
- Exit codes: Return 0 on success, 1 on error; use `exit 1` for fatal errors
- Use `set -e` in scripts to exit on first error

### String Handling & Redirects
- Always quote variables: `"${variable}"`
- Default values: `"${variable:-default}"`
- Use `printf "%s\n"` for output
- Redirects: `>| file` (clobber), `>> file` (append), `2>| /dev/null 1>&2`

### Code Organization
- Main entry: `src/gdl.sh`
- Common utils: `src/common/common-utils.sh`
- Core: `src/common/drive-utils.sh`, `src/common/download-utils.sh`, `src/common/auth-utils.sh`
- Parser: `src/common/flags.sh`, `src/common/parser.sh`, `src/common/gdl-common.sh`

### Helper Functions (reuse these)
- `_dirname` - Alternative to dirname
- `_bytes_to_human` - Convert bytes to human readable
- `_epoch` - Get epoch seconds
- `_count` - Count lines
- `_assert_regex` - Check regex
- `_set_value` - Set/export values (direct/indirect)
- `_trim` - Remove character from string

### Testing Notes
Tests use real Google Drive IDs. Use `./gdl --skip-internet-check "url_or_id" [flags]`
Test IDs: FILE_ID="14eh2_N3rGeGzUMamk2uyoU_CF9O7YUkA", DOCUMENT_ID="1Dziv2X5_UCMQ2weMI9duSUT6iayMikqRdoftJCwq_vg", FOLDER_ID="1AC0UsKfLZfflIkO7Ork78et5VzIvFSDM"

### Important: No Comments
Do NOT add comments to code unless absolutely necessary. Function documentation is sufficient.


<!-- CLAVIX:START -->
# Clavix Instructions for Generic Agents

This guide is for agents that can only read documentation (no slash-command support). If your platform supports custom slash commands, use those instead.

---

## ⛔ CLAVIX MODE ENFORCEMENT

**CRITICAL: Know which mode you're in and STOP at the right point.**

**OPTIMIZATION workflows** (NO CODE ALLOWED):
- Improve mode - Prompt optimization only (auto-selects depth)
- Your role: Analyze, optimize, show improved prompt, **STOP**
- ❌ DO NOT implement the prompt's requirements
- ✅ After showing optimized prompt, tell user: "Run `/clavix:implement --latest` to implement"

**PLANNING workflows** (NO CODE ALLOWED):
- Conversational mode, requirement extraction, PRD generation
- Your role: Ask questions, create PRDs/prompts, extract requirements
- ❌ DO NOT implement features during these workflows

**IMPLEMENTATION workflows** (CODE ALLOWED):
- Only after user runs execute/implement commands
- Your role: Write code, execute tasks, implement features
- ✅ DO implement code during these workflows

**If unsure, ASK:** "Should I implement this now, or continue with planning?"

See `.clavix/instructions/core/clavix-mode.md` for complete mode documentation.

---

## 📁 Detailed Workflow Instructions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Akianonymus/gdrive-downloader](https://github.com/Akianonymus/gdrive-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
