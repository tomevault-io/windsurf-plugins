---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

numd is a Nushell module for creating reproducible Markdown documents. It executes Nushell code blocks within markdown files and writes results back to the document.

## Common Commands

```nushell
# Run numd on a markdown file (updates file with execution results)
use numd; numd run README.md

# Preview mode (output to stdout, don't save)
use numd; numd run README.md --echo

# Dry run: return blocks that would execute (block_index, infostring, code), execute nothing
use numd; numd run README.md --dry-run

# Run tests (executes all example files and reports changes)
nu toolkit.nu test --json

# Clear outputs from a markdown file
use numd; numd clear-outputs path/to/file.md

# Strip markdown to get pure Nushell script
use numd; numd clear-outputs path/to/file.md --strip-markdown --echo
```

## Architecture

### Module Structure (`numd/`)

- **mod.nu**: Entry point exporting user-friendly commands (`run`, `clear-outputs`, etc.)
- **plumbing.nu**: Low-level pipeline commands for advanced usage/scripting
- **commands.nu**: Core implementation containing all command logic
- **doc.nu**: `doc` command rendering markdown docs for a module or command from `scope` data
- **parse-help.nu**: `parse-help` command for formatting --help output
- **parse.nu**: Frontmatter parsing utilities (`parse-frontmatter`, `to md-with-frontmatter`)

### Plumbing Commands

Low-level composable commands (import via `use numd/plumbing.nu`):

```nushell
use numd/plumbing.nu

# Parse markdown file into blocks table
plumbing parse-file file.md

# Strip output lines (# =>) from blocks
plumbing parse-file file.md | plumbing strip-outputs

# Execute code blocks and update with results
plumbing parse-file file.md | plumbing execute-blocks --save-intermed-script temp.nu

# Render blocks table back to markdown
plumbing parse-file file.md | plumbing strip-outputs | plumbing to-markdown

# Extract pure Nushell script (no markdown)
plumbing parse-file file.md | plumbing strip-outputs | plumbing to-numd-script
```

The high-level commands use these internally:
- `run` = `parse-file | execute-blocks | to-markdown`
- `clear-outputs` = `parse-file | strip-outputs | to-markdown`

### Core Processing Pipeline (in `commands.nu`)

1. **`parse-markdown-to-blocks`**: Parses markdown into a table classifying each block by type (`text`, ` ```nushell `, ` ```output-numd `) and action (`execute`, `print-as-it-is`, `delete`)

2. **`decorate-original-code-blocks`** + **`generate-intermediate-script`**: Transforms executable code blocks into a temporary `.nu` script with markers for output capture

3. **`execute-intermediate-script`**: Runs the generated script in a new Nushell instance, capturing stdout

4. **`extract-block-index`** + **`merge-markdown`**: Parses execution results using `#code-block-marker-open-N` markers and merges them back into the original markdown structure

### Fence Options

Blocks support fence options (e.g., ` ```nushell try, no-output `):
- `no-run`: Skip execution
- `no-output`: Execute but hide output
- `try`: Wrap in try-catch
- `new-instance`: Execute in separate Nushell instance
- `separate-block`: Output results in separate code block instead of inline `# =>`
- `run-once`: Execute code block once, then set to `no-run`

### Output Format Conventions

- Code blocks are split by blank lines (double newlines) into command groups
- Each command group is executed separately via `split-by-blank-lines`
- Lines starting with `# =>` contain output from previous command group
- Plain `#` comments are preserved; `# =>` output lines are regenerated on each run
- Use `separate-block` fence option to output results in a separate code block instead of inline `# =>`

## Testing

```nushell
# Run all tests (unit + integration)
nu toolkit.nu test

# Run only unit tests (nutest-based, tests internal functions)
nu toolkit.nu test-unit

# Run only integration tests (executes example markdown files)
nu toolkit.nu test-integration
```

**Output mode is auto-detected.** When stdout is a terminal you get the human view — only the non-passing tests (with the assertion message on failure), then a `N passed, M failed` summary. When stdout is piped or redirected (agents, CI) you get machine-readable JSON instead. This uses `is-terminal --stdout`, not `$nu.is-interactive` (which is false for any `nu toolkit.nu ...` script run and so cannot tell agent from human).

```nushell
nu toolkit.nu test --json    # force JSON even on a terminal
nu toolkit.nu test --pretty  # force the human view even when piped
nu toolkit.nu test --all     # human view: also list passing tests
```

JSON rows use a flat schema — `{type, name, status: 'passed'|'failed'|'changed', file, message}`. Note `status` (not nutest's `PASS`/`FAIL` `result` column); `message` holds the assertion text on failure, `null` otherwise. The JSON channel always carries every row — the failures-only trimming applies to the human view only.

### Unit Tests (`tests/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nushell-prophet/numd](https://github.com/nushell-prophet/numd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
