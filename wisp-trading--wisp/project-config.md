---
trigger: always_on
description: This project uses a centralized styling system located in `internal/ui/styles.go`. All TUI components MUST use these centralized styles to ensure consistency and maintainability.
---

# GitHub Copilot Instructions for WISP TUI Development

## TUI Styling Architecture

This project uses a centralized styling system located in `internal/ui/styles.go`. All TUI components MUST use these centralized styles to ensure consistency and maintainability.

### Mandatory Rules

#### 1. **NEVER Create Inline Styles**
❌ **FORBIDDEN:**
```go
titleStyle := lipgloss.NewStyle().
    Foreground(lipgloss.Color("#00D9FF")).
    Bold(true)
```

✅ **REQUIRED:**
```go
import "github.com/wisp-trading/wisp/internal/ui"

title := ui.TitleStyle.Render("My Title")
```

#### 2. **NEVER Hardcode Colors**
❌ **FORBIDDEN:**
```go
lipgloss.Color("#00D9FF")  // Direct hex color
lipgloss.Color("#7C3AED")  // Hardcoded purple
```

✅ **REQUIRED:**
```go
ui.ColorPrimary    // Cyan
ui.ColorSecondary  // Purple
ui.ColorSuccess    // Green
ui.ColorWarning    // Orange
ui.ColorDanger     // Red
ui.ColorMuted      // Gray
```

#### 3. **Use Existing Styles First**
Before creating any new style, check if an existing style can be used or adapted:
- `ui.TitleStyle` - Main view titles
- `ui.TitleCenteredStyle` - Centered dialog titles
- `ui.SectionHeaderStyle` - Section headers
- `ui.SubtitleStyle` - Muted secondary text
- `ui.BoxStyle` - Standard container box
- `ui.MenuBoxStyle` - Menu and dialog boxes
- `ui.DetailBoxStyle` - Detail information displays
- `ui.ErrorBoxStyle` - Error messages
- `ui.ItemStyle` - Unselected list items
- `ui.SelectedItemStyle` - Selected list items with cursor
- `ui.LabelStyle` - Form field labels
- `ui.ValueStyle` - Form field values
- `ui.DescriptionStyle` - Descriptive text
- `ui.CommandStyle` - Command names in help
- `ui.KeyHintStyle` - Keyboard shortcut hints
- `ui.StatusReadyStyle` - Success/enabled states
- `ui.StatusRunningStyle` - Warning/active states
- `ui.StatusDangerStyle` - Error/critical states
- `ui.StatusDisabledStyle` - Disabled/inactive states
- `ui.NetworkBadgeStyle` - Network indicators (mainnet)
- `ui.NetworkBadgeWarningStyle` - Testnet badges
- `ui.PnLProfitStyle` - Positive profit values
- `ui.PnLLossStyle` - Negative loss values
- `ui.PnLNeutralStyle` - Zero PnL values

#### 4. **Style Modification Pattern**
If you need to modify an existing style, use `.Copy()`:

✅ **CORRECT:**
```go
customBox := ui.BoxStyle.Copy().Width(50).Padding(2, 4)
```

❌ **WRONG:**
```go
customBox := lipgloss.NewStyle().
    Border(lipgloss.RoundedBorder()).
    BorderForeground(ui.ColorPrimary).
    Width(50)  // Duplicating BoxStyle definition
```

#### 5. **Use Utility Functions**
For common rendering patterns, use the utility functions in `ui`:

✅ **CORRECT:**
```go
pnlDisplay := ui.FormatPnL(totalPnL)  // Auto-styles positive/negative
progressBar := ui.RenderProgressBar(0.75, 50)  // Consistent progress bars
```

❌ **WRONG:**
```go
// Manually implementing PnL formatting
var pnlText string
if pnl > 0 {
    pnlText = lipgloss.NewStyle().
        Foreground(lipgloss.Color("#10B981")).
        Render(fmt.Sprintf("+$%.2f", pnl))
}
```

### Domain-Specific Styles

Domain-specific styles (e.g., monitor tabs, trading views) should:
1. Be located in their respective domain packages (e.g., `internal/handlers/strategies/monitor/styles.go`)
2. **Extend** base `ui` styles, not duplicate them
3. Include a comment explaining why domain-specific styling is needed

✅ **CORRECT - Domain Extension:**
```go
// monitor/styles.go
package monitor

import "github.com/wisp-trading/wisp/internal/ui"

// TabActiveStyle extends base styles for active tab indication
var TabActiveStyle = lipgloss.NewStyle().
    Foreground(ui.ColorPrimary).
    Bold(true).
    Padding(0, 2).
    Underline(true)  // Domain-specific: tab underline

// Use base formatter for PnL
var FormatPnL = ui.FormatPnL
```

❌ **WRONG - Duplicate Definition:**
```go
// Duplicating what's already in ui package
var ProfitStyle = lipgloss.NewStyle().
    Foreground(ui.ColorSuccess).
    Bold(true)
```

### Adding New Styles

When adding a new style to `internal/ui/styles.go`:

1. **Check for duplicates** - ensure the style doesn't already exist
2. **Follow naming convention**: `[Domain][Element][Variant]Style`
   - Examples: `TitleCenteredStyle`, `NetworkBadgeWarningStyle`, `StatusDisabledStyle`
3. **Add documentation** - explain when to use the style
4. **Group logically** - place with related styles (titles with titles, status with status)
5. **Request review** - new base styles should be reviewed to avoid bloat

**Example of proper style addition:**
```go
// ErrorBoxStyle is used for displaying error messages prominently
// Use this for critical errors that need user attention
ErrorBoxStyle = lipgloss.NewStyle().
    Foreground(ColorDanger).
    Bold(true).
    Border(lipgloss.RoundedBorder()).
    BorderForeground(ColorDanger).
    Padding(1, 2)
```

### Testing Style Changes

When modifying `internal/ui/styles.go`:
1. Search for all usages of the modified style
2. Test affected views visually
3. Ensure backward compatibility
4. Update this document if changing style semantics

### Code Review Checklist

Before submitting a PR with TUI changes:
- [ ] No inline `lipgloss.NewStyle()` calls outside `internal/ui/styles.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wisp-trading/wisp](https://github.com/wisp-trading/wisp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
