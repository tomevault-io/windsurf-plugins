---
trigger: always_on
description: - All of comments in code base, commit message, PR content and title should be written in English.
---

## Important Restriction
- All of comments in code base, commit message, PR content and title should be written in English.
  - If you find any Korean text, please translate it to English.
- **UI Language**
  - All user-facing text in the app MUST be in English.
- Default review language: English.

### Obey design decisions that specifically decided before by me
- All the decision is speicifed in `@AGENTS-design-decisions.md` file.

## Coding Rules

### Branding Guides
- Official Brand: `OpenCode Bar`
  - Don't use mis-capitalized or malformed forms like `Opencode Bar` or versions without the space (e.g. `OpenCodeBar`).
- File Name:
  - `OpenCode Bar` if it allows whitespace.
  - `OpenCode-Bar` if it allows dash.
  - `opencode-bar` (all lowercase) if the environment prefers lowercase file names.
  - `opencodebar` (all lowercase, no separators) if it doesn't allow whitespace or dashes.
- Bundle ID: `com.copilotmonitor.CopilotMonitor`

### UI Styling Rules
- **No colors for text emphasis**: Do NOT use `NSColor` attributes like `.foregroundColor` for menu items or labels.
- **DO NOT USE SPACES TO ALIGN TEXT**: Don't use spaces like "   Words:" to align the spacing.
- **Use instead**:
  - **Bold**: `NSFont.boldSystemFont(ofSize:)` for important text
  - **Underline**: `.underlineStyle: NSUnderlineStyle.single.rawValue` for critical warnings
  - **SF Symbols**: Use `NSImage(systemSymbolName:accessibilityDescription:)` for menu item icons
  - **Emphasis**: Use system colors like secondaryLabelColor and etc.
  - **Offset**: Use offset for aligning text with other items and lines. Use the below constants.
- **Do NOT use**:
  - **Emoji**: Never use emoji for menu item icons. Always use SF Symbols instead.
  - **RGB color**: Use only pre-defined colors by system (systemGreen, systemOrange, and etc) to consider dark/light mode compatiblity.
- **Exception**:
  - While you can't use color for text, progress bars and status indicators can use system color.
  - You can use color for text which is right-aligned text only.
- Others
  - **Never use random spaces for separating label**
    - BEST:
      - Left: "OpenRouter"
      - Right: "$37.42" - Additional Custom View Label on the right with right-aligned text (by offset calculating)
    - OK: "OpenRouter: $37.42"
    - OK: "OpenRouter ($37.42)"
    - NO: "OpenRouter    $37.42" (stupid random spaces)
  - **USD**
    - Use only two decimals when expressing dollars. (e.g. `$00.00`) 

### Explicit 'used' or 'left'
- To avoid confusing of used % or left %, explicit if it's used or left on every labels.

### Menu Item Layout Constants (MUST follow strictly)
All custom menu item views MUST use `MenuDesignToken` from `Helpers/MenuDesignToken.swift`:
```swift
// Usage examples
let width = MenuDesignToken.Dimension.menuWidth      // 300
let height = MenuDesignToken.Dimension.itemHeight    // 22
let fontSize = MenuDesignToken.Dimension.fontSize    // 13
let iconSize = MenuDesignToken.Dimension.iconSize    // 16
let geminiIconSize = MenuDesignToken.Dimension.geminiIconSize // 17

let leading = MenuDesignToken.Spacing.leadingOffset      // 14
let leadingIcon = MenuDesignToken.Spacing.leadingWithIcon // 36
let trailing = MenuDesignToken.Spacing.trailingMargin    // 14

let font = MenuDesignToken.Typography.defaultFont
let mono = MenuDesignToken.Typography.monospacedFont
let bold = MenuDesignToken.Typography.boldFont

let rightX = MenuDesignToken.rightElementX  // 270 (computed)
```
- **NEVER** hardcode pixel values - always use `MenuDesignToken`
- **ALWAYS** reuse `createDisabledLabelView()` when possible instead of creating custom NSView
- When adding new constants, add them to `MenuDesignToken.swift` first, then update this section

### Pre-Development Setup (MUST run before any work)
Before starting any development work, run:
```bash
make setup
```
This sets up git hooks for automated pre-commit checks:
- **SwiftLint**: Validates Swift code style
- **action-validator**: Validates GitHub Actions workflow YAML files

### Build & Run Commands
Use the VSCode task "🐛 Debug: Kill + Build + Run" for development.
- Automatically detects DerivedData path via `xcodebuild -showBuildSettings`
- Works with multiple worktrees (each has its own DerivedData directory)

```bash
# Watch logs
log stream --predicate 'subsystem == "com.opencodeproviders"' --level debug
# or check file: cat /tmp/provider_debug.log
```

### Instruction of each task
- In all changes, always write debugging log for actually printing before you confirming the feature is fully functional.
- After each change, follow:
  - Clear cache and compile the binary
  - Kill the existing process, and run the new app.
  - Confirm if it works through **logs**.

## Release Policy
- **Workflow**: STRICTLY follow `docs/RELEASE_WORKFLOW.md` for versioning, building, signing, and notarizing.
- **Signing**: All DMGs distributed via GitHub Releases **MUST** be signed with Developer ID and **NOTARIZED** to pass macOS Gatekeeper.
- **Documentation**: Update `README.md` and screenshots if UI changes significantly before release.

### CI Universal Binary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opgginc/opencode-bar](https://github.com/opgginc/opencode-bar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
