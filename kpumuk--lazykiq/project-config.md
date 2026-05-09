---
trigger: always_on
description: Bubble Tea TUI for Sidekiq monitoring. Go 1.26.
---

# Lazykiq - Agents Context

## What

Bubble Tea TUI for Sidekiq monitoring. Go 1.26.

## Git

- Use flat branch names. Do not prefix with `feature/`, `perf/`, `fix/`, etc. unless explicitly requested.

## Code Quality

- Make sure to always perform `mise run ci` to ensure code formatting matches the style of the repository, linters don't have any offenses, and tests pass.

```bash
mise run ci   # run formatter, tests, linter
mise run fmt  # format code
mise run test # run tests
mise run lint # run all linters
# only focus on modernize linter
mise run lint --enable-only modernize
# address all auto-correctable issues
mise run lint --enable-only modernize --fix
```

## Structure

```text
cmd/lazykiq/main.go          - CLI entry point
internal/cmd/root.go         - cobra/fang CLI wiring, runs UI
internal/
  sidekiq/
    client.go                - Redis client, stats + busy/queues APIs
    dashboard.go             - Redis INFO + stats history/realtime helpers
    metrics.go               - Sidekiq Pro metrics rollups + histograms
    job.go, queue.go          - job + queue models/parsers
    sorted.go                - dead/retry/scheduled sorted-set helpers
  ui/
    app.go                   - main model, view stack + stackbar, tick + RefreshMsg, error overlay
    keys.go                  - KeyMap struct, DefaultKeyMap()
    theme/theme.go           - Theme (Dark/Light), Styles, NewStyles(), stackbar styles
    components/
      errorpopup/            - centered overlay for connection errors
      filterinput/           - /-activated filter input + ActionMsg
      frame/                 - titled bordered box with optional meta (replaces renderBorderedBox)
      jsonview/              - syntax-highlighted JSON renderer
      messagebox/            - centered empty-state box
      metrics/               - top bar: Processed|Failed|Busy|Enqueued|Retries|Scheduled|Dead
      navbar/                - bottom bar: view keys + quit
      stackbar/              - breadcrumb for stacked views
      table/                 - reusable scrollable table with selection
    format/format.go         - Duration, Bytes, Args, Number formatters
    views/
      view.go                - View interface + msgs + Disposable/Setter interfaces
      dashboard.go, queues.go, busy.go, retries.go, scheduled.go, dead.go
      errors_summary.go, errors_details.go, errors_data.go
      jobdetail.go           - stacked job detail view
```

## Patterns

- Views implement `views.View` interface
- Components take `*theme.Styles`, have SetStyles()
- Theme uses AdaptiveColor; no runtime toggle
- All color values must live in `theme.DefaultTheme`; no inline colors outside it (dashboard charts are the only temporary exception while ntcharts uses lipgloss v1)
- Border title/meta: use `components/frame` (title is on the top border line)
- No emojis in UI except strategically placed glyphs from Nerd Font, such as "⚙" - keep text clean and professional
- Shared components: no lipgloss.NewStyle() calls in render paths - pass all styles via struct/DefaultStyles
- Table in `components/table/` subpackage to avoid import cycle (components ↔ views)
- Table: last column not truncated/padded to allow horizontal scroll of variable content
- Empty states: prefer `messagebox` for centered messages
- Prefer `mathutil.Clamp` for bounds clamping instead of nested `min/max` or manual if ladders
- Detail views are stacked: emit `ShowJobDetailMsg`/`ShowErrorDetailsMsg` and let `app.go` push views
- Views with transient state should implement `views.Disposable` for cleanup when popped
- App keeps a view registry + stack; `viewOrder` drives navbar ordering and view hotkeys

## Testing

Use **hybrid approach**: property tests (behavior) + golden tests (visual regression).

### Property Tests

- Test behaviors using table-driven tests: `map[string]struct{ ... }`
- Check: dimensions respected, content present, edge cases (zero dimensions, disabled items, truncation)
- Use `strings.Contains()` for presence checks, avoid brittle position assertions

### Golden Tests

- Prefix with `TestGolden*`
- Capture exact visual output to catch misalignment/spacing changes
- Strip ANSI before comparison: `output := ansi.Strip(m.View()); golden.RequireEqual(t, []byte(output))`
- Update after intentional visual changes: `go test ./path -update`
- Cover: empty state, single column, combined layout, different widths, real-world examples

**Rule:** Property tests alone miss visual regressions (misalignment, spacing). Always add golden tests for UI components with layout requirements.

## Component Pattern (bubbles-style)

Follow the charmbracelet/bubbles pattern for reusable components:

```go
// 1. Styles struct - exported, holds all styles
type Styles struct {
    Title  lipgloss.Style
    Border lipgloss.Style
}

// 2. DefaultStyles() - returns sensible defaults
func DefaultStyles() Styles {
    return Styles{
        Title:  lipgloss.NewStyle().Bold(true),
        Border: lipgloss.NewStyle(),
    }
}

// 3. Model struct - holds all state
type Model struct {
    styles  Styles  // unexported, use SetStyles()
    width   int
    height  int
    // ... other state
}

// 4. Option type for functional options
type Option func(*Model)

// 5. New() constructor with functional options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kpumuk/lazykiq](https://github.com/kpumuk/lazykiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
