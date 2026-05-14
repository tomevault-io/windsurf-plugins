---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `gemistat`, a wrapper for [gemini-cli](https://github.com/google-gemini/gemini-cli) that tracks token usage and costs in real-time. The project uses Bun as the runtime and TypeScript for type safety.

## Guide for lsmcp mcp

You are a professional coding agent concerned with one particular codebase. You have
access to semantic coding tools on which you rely heavily for all your work, as well as collection of memory
files containing general information about the codebase. You operate in a frugal and intelligent manner, always
keeping in mind to not read or generate content that is not needed for the task at hand.

When reading code in order to answer a user question or task, you should try reading only the necessary code.
Some tasks may require you to understand the architecture of large parts of the codebase, while for others,
it may be enough to read a small set of symbols or a single file.
Generally, you should avoid reading entire files unless it is absolutely necessary, instead relying on
intelligent step-by-step acquisition of information. Use the symbol indexing tools to efficiently navigate the codebase.

IMPORTANT: Always use the symbol indexing tools to minimize code reading:

- Use `search_symbol_from_index` to find specific symbols quickly (after indexing)
- Use `get_document_symbols` to understand file structure
- Use `find_references` to trace symbol usage
- Only read full files when absolutely necessary

You can achieve intelligent code reading by:

1. Using `index_files` to build symbol index for fast searching
2. Using `search_symbol_from_index` with filters (name, kind, file, container) to find symbols
3. Using `get_document_symbols` to understand file structure
4. Using `get_definitions`, `find_references` to trace relationships
5. Using standard file operations when needed

## Working with Symbols

Symbols are identified by their name, kind, file location, and container. Use these tools:

- `index_files` - Build symbol index for files matching pattern (e.g., '\*_/_.ts')
- `search_symbol_from_index` - Fast search by name, kind (Class, Function, etc.), file pattern, or container
- `get_document_symbols` - Get all symbols in a specific file with hierarchical structure
- `get_definitions` - Navigate to symbol definitions
- `find_references` - Find all references to a symbol
- `get_hover` - Get hover information (type signature, documentation)
- `get_diagnostics` - Get errors and warnings for a file
- `get_workspace_symbols` - Search symbols across the entire workspace

Always prefer indexed searches (tools with `_from_index` suffix) over reading entire files.

## Development Commands

- `bun run start` - Run the main application (wrapper for gemini-cli)
- `bun run typecheck` - Type check TypeScript files using tsgo
- `bun run lint` - Run ESLint to check code quality
- `bun run format` - Run ESLint with --fix to format code
- `bun run test` - Run vitest tests (supports in-source testing with global imports)

## Running the Application

The main entry point is `index.ts` which acts as a wrapper around gemini-cli:

```bash
# Run directly
bun run index.ts

# Or with arguments (any gemini-cli arguments work)
bun run index.ts --model gemini-2.0-flash-exp
./index.ts chat
```

## Configuration

The application supports environment variables for configuring output directories:

- `GEMISTAT_OUTPUT_DIR` - Base directory to save telemetry files (default: `~/.gemini/usage`)

File Structure:

- Files are organized as `{GEMISTAT_OUTPUT_DIR}/{YYYY-MM-DD}/uuid.jsonl`
- Each gemini-cli session creates a unique UUID-based filename within a date directory
- This prevents overwrites when running multiple sessions on the same day

Examples:

```bash
# Use default directory (~/.gemini/usage)
# Creates: ~/.gemini/usage/{YYYY-MM-DD}/uuid.jsonl
./index.ts chat

# Custom directory
# Creates: /tmp/gemini-logs/{YYYY-MM-DD}/uuid.jsonl
GEMISTAT_OUTPUT_DIR=/tmp/gemini-logs ./index.ts chat
```

## Architecture

The codebase follows a modular architecture with three main components:

### Core Files

- `index.ts` - Main entry point that spawns gemini-cli with telemetry enabled
- `src/telemetry-watcher.ts` - Watches OpenTelemetry JSONL files and emits events
- `src/stats-display.ts` - Manages real-time display of token usage and costs
- `src/pricing.ts` - Fetches and caches model pricing data from LiteLLM

### Key Concepts

1. **Telemetry Capture**: Uses gemini-cli's `--telemetry-outfile` to capture OpenTelemetry events in JSONL format
2. **Event Processing**: `TelemetryWatcher` parses telemetry events and emits structured events for token usage and API responses
3. **Real-time Display**: `StatsDisplay` maintains running totals and shows live statistics during gemini-cli sessions
4. **Cost Calculation**: Fetches pricing data from LiteLLM's database and calculates costs for different token types (input, output, cached, thoughts, tools)

### Event Flow

```
gemini-cli → telemetry file → TelemetryWatcher → events → StatsDisplay → console output
```

## Output Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryoppippi/gemistat](https://github.com/ryoppippi/gemistat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
