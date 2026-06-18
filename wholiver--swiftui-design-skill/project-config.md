---
trigger: always_on
description: SwiftUI frontend visual design skill. Creates beautiful, distinctive iOS/macOS interfaces that avoid generic AI slop patterns. Covers design direction, layout systems, typography, color, spacing, brand integration, and design review. Use when designing new SwiftUI views, reviewing UI quality, creating iOS prototypes, choosing visual styles, improving app aesthetics, or when the UI looks generic or AI-generated.
---


# SwiftUI Design Skill

> **TL;DR**: Design SwiftUI apps that look intentionally crafted, not AI-generated. Follow the 6 Anti-Slop Rules, use the Design Direction workflow when requirements are vague, and always pass the 5-Dimension Review before shipping.

## When to Use

- Designing new SwiftUI views or screens from scratch
- Reviewing or improving existing UI quality
- Creating iOS app prototypes or mockups
- Choosing a visual style or design direction
- Integrating brand assets into SwiftUI
- When someone says "the UI looks generic" or "make it look better"
- When building a new feature that needs visual design

## When NOT to Use

- Pure backend / API work with no UI
- Fixing SwiftUI compilation errors (use swiftui-expert-skill instead)
- Performance optimization of existing views (use swiftui-expert-skill)
- Accessibility compliance only (use swiftui-expert-skill's accessibility topic)

## Prerequisites

- Xcode 16+ with iOS 18+ / macOS 15+ SDK
- SwiftUI framework (not UIKit unless explicitly requested)
- Familiarity with SwiftUI view hierarchy and modifiers

---

## Core Design Philosophy

These 6 principles guide every design decision. They are distilled from real-world design practice and anti-patterns observed in AI-generated UI.

### Principle 1: Start From Context, Never Blank

Before designing any screen, ask:
- Is there an existing design system, UI kit, or Figma file?
- Is there an existing codebase with established visual language?
- Are there brand guidelines or a style guide?

If no context exists, create a **brand-spec.md** first (see Brand Asset Protocol below).

### Principle 2: Junior Designer Mode — Hypothesize Early, Iterate Fast

Don't wait for the "perfect" design. Show hypotheses with placeholders early:
- Gray boxes for images are **better** than bad AI-generated SVGs
- System fonts with good spacing > custom fonts with bad spacing
- A rough but honest layout > a polished but generic one

### Principle 3: Offer Variations, Not Finals

Always present **2-3 distinct design directions**, not one "final answer." Each variation should:
- Represent a different design philosophy (e.g., editorial, functional, expressive)
- Have a clear rationale for why it works
- Be visually distinct — not just color swaps

### Principle 4: Placeholder > Bad Implementation

If you cannot source a real asset (icon, image, illustration):
- Use a clean gray placeholder with clear labeling
- Never use AI-generated SVG clipart
- Never use emoji as icon substitutes in production UI
- For icons, prefer SF Symbols (Apple's built-in icon library)

### Principle 5: System First, Don't Fill

Every element must earn its place. Before adding any UI element, ask:
- Does this serve the user's goal?
- Can this be combined with something else?
- What happens if I remove it?

Whitespace is a feature, not emptiness.

### Principle 6: Anti-AI-Slop

These patterns are **banned** in production UI. See `references/anti-ai-slop.md` for detailed rules.

**Banned visual patterns:**
- ❌ Purple-to-blue gradient backgrounds
- ❌ Emoji as functional icons
- ❌ Rounded cards with left-border accent color
- ❌ Generic "hero section" with centered text over gradient
- ❌ Inter/Roboto as display fonts on Apple platforms
- ❌ Neon-on-dark "cyber" aesthetic (#0D1117 base)
- ❌ Symmetric 3-column feature grids
- ❌ AI-drawn SVG illustrations or CSS silhouettes

**What to do instead:**
- ✅ Single warm accent color on neutral background
- ✅ SF Symbols for icons, or custom icon set
- ✅ Serif display font (e.g., New York, Georgia) for headings
- ✅ Restrained info density — let content breathe
- ✅ One signature detail at 120% effort per screen
- ✅ Real photography or clean placeholders

---

## Brand Asset Protocol

When building UI for a specific brand, follow this 5-step hard flow:

1. **Ask** — Request brand guidelines, logo files, color palette, typography
2. **Search** — If not provided, search the web for "[brand name] brand guidelines"
3. **Download** — Get actual asset files (PNG/SVG logos, font files)
4. **Verify** — Confirm colors match official sources (check hex values)
5. **Write brand-spec.md** — Document: primary/secondary colors, fonts, logo usage rules, spacing tokens

**Quality threshold**: 5 real brand colors, 10 verified design tokens, 2 font families, 8pt spacing grid.

If no brand exists, create a brand-spec.md with:
- 1 primary accent color (warm, not purple-blue gradient)
- 1 neutral palette (grays with warm undertone)
- 1 display font + 1 body font (SF Pro default is fine)
- 8pt spacing grid

---

## Design Direction Workflow

When requirements are vague or the user asks "what style should we use?", follow this workflow:

### Phase 1: Understand Context (5 min)
1. Ask: What is the app? Who is the user? What platform (iOS/macOS/watchOS)?
2. Ask: Any reference apps or websites they admire?
3. Check: Existing design system or brand assets?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wholiver/swiftui-design-skill](https://github.com/Wholiver/swiftui-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
