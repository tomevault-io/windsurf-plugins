---
trigger: always_on
description: Design principles for building polished, native-feeling SwiftUI apps and widgets. Use this skill when creating or modifying SwiftUI views, iOS widgets (WidgetKit), or any native Apple UI. Ensures proper spacing, typography, colors, and widget implementations that look and feel like quality apps rather than AI-generated slop.
---


This skill encodes design principles derived from comparing polished, production-quality SwiftUI apps against poorly-built ones. The patterns here represent what separates an app that feels "right" from one where the margins, spacing, and text sizes just look "off."

Apply these principles whenever building or modifying SwiftUI interfaces, WidgetKit widgets, or any native Apple UI.

## Core Philosophy

**Restraint over decoration.** Every pixel must earn its place. A polished app uses fewer colors, fewer font sizes, fewer spacing values, and fewer words — but uses them consistently. Over-engineering visual elements (custom gradients, decorative borders, bespoke dividers) creates visual noise. Native components and system colors create harmony.

**Attention is scarce.** Keep UI copy shorter than you think it needs to be. Prefer one clear headline and one compact supporting block over repeated explanation in the title, subtitle, body, and footer. If a screen needs rationale, put it in one purposeful place instead of scattering it across the page.

---

## 1. Spacing System: Use a Consistent Grid

**CRITICAL**: Use spacing values from a base-4/base-8 grid. Never use arbitrary values.

### Allowed spacing values
```
4, 8, 12, 16, 20, 24, 32, 40, 48
```

### Bad (arbitrary values that create visual dissonance)
```swift
// WRONG - these numbers have no relationship to each other
.padding(.bottom, 26)
.padding(.bottom, 34)
.padding(.bottom, 36)
HStack(spacing: 18)
.padding(14)
```

### Good (values from a consistent grid)
```swift
// RIGHT - predictable rhythm the eye can follow
.padding(.horizontal, 20)
.padding(.top, 8)
Spacer().frame(height: 32)
HStack(spacing: 4)  // or 8, 12, 16
.padding(.vertical, 12)
.padding(.horizontal, 16)
```

### Standard padding assignments
- **Outer content padding**: 16-20pt horizontal
- **Between major sections**: 24-32pt vertical
- **Within grouped components**: 4-12pt
- **Card/row internal padding**: 12-16pt vertical, 16pt horizontal

---

## 2. Typography: Hierarchy Through Weight, Not Just Size

### The principle
Use **fewer font sizes** with **clear weight differentiation**. Lighter weights at larger sizes; medium/regular at smaller sizes. This creates sophistication rather than visual chaos.

### Recommended type scale (for a data-focused app)
| Role | Size | Weight | Notes |
|------|------|--------|-------|
| Hero number | 36-42pt | `.light` | Large but visually light -- elegant, not heavy |
| Secondary stat | 20-24pt | `.light` | Same weight family as hero, smaller |
| Body / toggle label | 15pt | `.regular` | Standard iOS body size |
| Section header (uppercase) | 11pt | `.medium` | With tracking/letter-spacing |
| Caption / subtitle | 11-13pt | `.regular` | Secondary information |

### Bad (too many sizes, inconsistent weights)
```swift
// WRONG - 7 different sizes with no clear system
.font(.system(size: 60, weight: .ultraLight))   // hero
.font(.system(size: 44, weight: .regular))        // stat (too close to hero)
.font(.system(size: 31, weight: .ultraLight))     // percent symbol (odd ratio)
.font(.system(size: 18, weight: .regular))        // label (too big for a toggle)
.font(.system(size: 14, weight: .regular))        // header
.font(.system(size: 13, weight: .regular))        // another header
.font(.system(size: 12, weight: .regular))        // button (too small to read)
```

### Good (clear hierarchy, fewer sizes)
```swift
// RIGHT - 5 sizes, clear purpose for each
.font(.system(size: 42, weight: .light, design: .monospaced))    // hero
.font(.system(size: 24, weight: .light, design: .monospaced))    // stat value
.font(.system(size: 15, weight: .regular, design: .monospaced))  // body
.font(.system(size: 14, weight: .regular, design: .monospaced))  // secondary
.font(.system(size: 11, weight: .medium, design: .monospaced))   // label
```

### Font design consistency
Pick ONE font design and use it everywhere -- app AND widgets:
```swift
// If using monospaced, use it everywhere
design: .monospaced  // app views, widgets, lock screen -- all of them

// NEVER mix designs between app and widgets
// BAD: .monospaced in app, .rounded in lock screen widget
```

### Letter spacing (tracking)
Use at most 2 values, and only on uppercase labels:
```swift
.tracking(1.5)  // section labels: "NOTIFICATIONS", "DAY", "LEFT"
.tracking(3)    // navigation/toolbar titles
```

**Never use 3+ different tracking values** like `kerning(4)`, `kerning(4.5)`, `kerning(5)` -- the differences are imperceptible but the inconsistency registers subconsciously.

### Numeric formatting for identifiers
Years and other fixed identifiers should not be locale-grouped.
```swift
// RIGHT - stable, non-grouped identifier text
Text(String(year))                  // "2026"
Text(year, format: .number.grouping(.never))

// WRONG - locale grouping can render "2,026"
Text("\(year)")
```

---

## 3. Colors: System Semantic Colors Over Hardcoded Values

### The principle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arjitj2/swiftui-design-principles](https://github.com/arjitj2/swiftui-design-principles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
