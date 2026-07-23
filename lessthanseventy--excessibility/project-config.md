---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Excessibility** is an Elixir library for accessibility snapshot testing in Phoenix applications. It captures HTML snapshots during tests and runs them through Pa11y for WCAG compliance checking.

## Development Commands

### Testing
```bash
# Run all tests
mix test

# Run specific test file
mix test test/snapshot_test.exs

# Run tests with a specific line number
mix test test/snapshot_test.exs:42

# Run tests in watch mode (interactive)
mix test.interactive
```

### Linting & Formatting
```bash
# Run static analysis
mix credo

# Format code (using Styler)
mix format
```

### Documentation
```bash
# Generate documentation
mix docs

# View docs locally
open doc/index.html
```

### Accessibility Testing
```bash
# Run Pa11y on all existing snapshots
mix excessibility

# Run specific test + Pa11y on new snapshots
mix excessibility test/my_test.exs
mix excessibility test/my_test.exs:42
mix excessibility --only a11y

# Approve pending diffs interactively
mix excessibility.approve

# Approve all diffs as good (keep baseline)
mix excessibility.approve --keep good

# Approve all diffs as bad (accept new versions)
mix excessibility.approve --keep bad
```

## Claude Workflow for Accessibility

When helping users write accessible Phoenix code, follow this workflow:

### 1. Add Snapshot Calls to Tests

For any LiveView or controller test, add `html_snapshot()` calls to capture rendered HTML:

```elixir
defmodule MyAppWeb.PageLiveTest do
  use MyAppWeb.ConnCase
  use Excessibility  # Required

  test "page is accessible", %{conn: conn} do
    {:ok, view, _html} = live(conn, "/")

    # Capture snapshot for Pa11y
    html_snapshot(view)

    # Continue with assertions...
  end
end
```

### 2. Run Tests to Generate Snapshots

```bash
mix test test/my_app_web/live/page_live_test.exs
```

This creates HTML files in `test/excessibility/html_snapshots/`.

### 3. Run Pa11y Accessibility Check

```bash
# Check all existing snapshots
mix excessibility

# Or run a specific test and check new snapshots in one command
mix excessibility test/my_app_web/live/page_live_test.exs
mix excessibility test/my_test.exs:42
mix excessibility --only a11y
```

Pa11y will report WCAG violations. Common issues:
- Missing form labels
- Low color contrast
- Missing alt text
- Invalid ARIA attributes

### 4. Fix Issues and Re-test

After fixing accessibility issues, re-run tests and Pa11y to verify.

### Timeline Analysis (for debugging)

Use `mix excessibility.debug` to analyze LiveView behavior. All arguments pass through to `mix test`:

```bash
# Run a test file with debug analysis
mix excessibility.debug test/my_live_view_test.exs

# Run specific test by line number
mix excessibility.debug test/my_live_view_test.exs:42

# Run tests with a tag
mix excessibility.debug --only live_view

# With debug options
mix excessibility.debug test/my_test.exs --analyze=memory
mix excessibility.debug test/my_test.exs --no-analyze
mix excessibility.debug test/my_test.exs --verbose
mix excessibility.debug test/my_test.exs --full
mix excessibility.debug test/my_test.exs --highlight=current_user,cart
```

This generates `timeline.json` with event flow, memory usage, and pattern analysis - useful for debugging performance but separate from accessibility testing.

**Available Analyzers (Default Enabled):**

- `memory` - Detects memory bloat and leaks using adaptive thresholds
- `performance` - Identifies slow events and bottlenecks
- `data_growth` - Analyzes list growth patterns
- `event_pattern` - Detects inefficient event patterns
- `ecto_query_analysis` - Full Ecto query analysis with N+1 detection
- `assign_diff` - Detects large assigns re-diffed over the wire
- `message_flooding` - Detects high-frequency handle_info patterns
- `state_machine` - Analyzes state transitions
- `render_efficiency` - Detects wasted renders with no state changes
- `assign_lifecycle` - Finds dead state (assigns that never change)
- `handle_event_noop` - Detects empty event handlers
- `form_validation` - Flags excessive validation roundtrips
- `summary` - Natural language timeline overview

**Available Analyzers (Opt-in):**

- `cascade_effect` - Detects rapid event cascades (use `--analyze=cascade_effect`)
- `hypothesis` - Root cause suggestions (use `--analyze=hypothesis`)
- `code_pointer` - Maps events to source locations (use `--analyze=code_pointer`)
- `accessibility_correlation` - Flags state changes with a11y implications (use `--analyze=accessibility_correlation`)
- `component_rerender` - Detects unnecessary component re-renders (use `--analyze=component_rerender`)
- `push_event_volume` - Detects excessive push_event volume to JS hooks (use `--analyze=push_event_volume`)

**Timeline Enrichments:**

Timeline events are automatically enriched with:
- `assign_sizes` - Per-assign byte sizes (map of assign name to bytes)
- `total_memory` - Total assigns memory in bytes
- `largest_assign` - Largest assign name and size tuple
- `event_duration_ms` - Event duration from telemetry
- `ecto_queries` - Ecto queries fired during event (list of query maps)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lessthanseventy/excessibility](https://github.com/lessthanseventy/excessibility) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
