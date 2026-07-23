---
trigger: always_on
description: Rational, Restrained, Warm, Unique
---

# Claude Style

Rational, Restrained, Warm, Unique

---

## Design Philosophy

### Brand Philosophy

**"AI with Warmth"**

Claude's design philosophy is injecting warmth into rational restraint. Not a cold technical tool, but an intelligent and empathetic assistant. Design conveys professionalism without losing approachability through warm ocher orange and warm white tones.

### Design Philosophy

1. **Restrained Minimalism - Function First**
   - Keep only core information and interactions
   - Remove redundant decoration, follow "less is more"
   - Visual elements serve function, don't steal the show

2. **Secure & Transparent - Auditable**
   - Interface clearly shows permissions, operation boundaries
   - Let users perceive security and controllability
   - Don't hide risks, don't blur operation logic

3. **Developer Friendly - Professional Quality**
   - Target technical audience, emphasize code feel
   - Strong logic, high professionalism
   - Calm and rational visual style, not flashy

4. **Warm & Unique - Differentiated**
   - Warm white tone distinguishes from cold development tools
   - Ocher orange brand color, unique and recognizable
   - Stand out among similar products

### Core Keywords

- Rational
- Restrained
- Warm
- Professional
- Trustworthy

---

## Overview

Claude style originates from Claude Code official VS Code theme—unique warm white tone + ocher orange accent. Suitable for developer tools, AI products.

---

## Color System

### Brand Color

```css
/* Claude Ocher Orange - Brand Primary Color */
--color-primary: #C15F3C;
--color-primary-hover: #A14A2F;
--color-primary-muted: #C15F3C80;
```

#### Usage Guidelines

**When to Use:**

- Status bar background (brand identity)
- Primary buttons (CTA)
- Selected states, active states
- Links and key interactive elements
- Code highlight keywords

**Avoid Using:**

- Large area backgrounds (ocher orange has high saturation)
- Non-critical action buttons
- Decorative elements

**Usage Principle:** Not exceeding 8% of page area

**Why Ocher Orange?**
- Warmth: Distinguishes from cold blue development tools
- Uniqueness: Stands out among similar products
- Professionalism: Ocher orange is warm yet steady
- Recognizability: Distinctive brand color, memorable

### Background Colors

```css
/* Light mode - Warm White */
--color-bg: #FAFAF9;        /* Main background */
--color-bg-editor: #FAF9F4;  /* Editor background */
--color-bg-secondary: #F5F4ED; /* Sidebar */
--color-bg-tertiary: #F0EFED; /* Secondary background */
--color-bg-elevated: #FDFCFB; /* Floating layer */

/* Dark mode */
--color-bg-dark: #171615;
--color-bg-secondary-dark: #1F1E1D;
--color-bg-tertiary-dark: #2A2928;
```

### Text Colors

```css
/* Light mode */
--color-text: #2C2C2C;       /* Primary text */
--color-text-secondary: #525252;
--color-text-tertiary: #A19A94;
--color-text-disabled: #706F6E;

/* Dark mode */
--color-text-dark: #FAF9F4;
--color-text-secondary-dark: #A7A8A7;
--color-text-tertiary-dark: #706F6E;
```

### Border Colors

```css
/* Light mode */
--color-border: #E8E6E3;
--color-border-hover: #D4D4D4;

/* Dark mode */
--color-border-dark: #47494B;
--color-border-subtle-dark: #2A2928;
```

#### Why Light Borders?

- Restrained design: Borders don't distract, content is primary
- Clear hierarchy: Distinguish areas through subtle borders
- Avoid clutter: Dark borders make interface appear harsh

### Accent Colors

```css
/* Claude Signature Accents */
--color-accent: #C15F3C;       /* Ocher - Primary accent */
--color-accent-gold: #EEC554;    /* Gold - Highlight/Special */
--color-accent-blue: #0366D6;
--color-accent-green: #28A745;
--color-accent-red: #D73A49;
--color-accent-pink: #EA4AAA;
--color-accent-cyan: #39C5CF;
```

#### Functional Color Guidelines

- Success: Green #28A745
- Warning: Gold #EEC554 (softer)
- Error: Red #D73A49 (low saturation)
- Info: Blue #0366D6

**Avoid High Saturation:** All functional colors use low saturation, maintaining restrained professionalism

---

## Interaction Philosophy

### Animation Principles

**Fast, Precise, Non-Intrusive**

Developer tools need fast response, animations cannot affect efficiency.

#### State Feedback

- Focus: Border highlight + glow (ocher orange)
- Hover: Slight background change
- Click: Slight scale (scale 0.98)

#### Transition Duration

- Fast: 100ms (button hover)
- Normal: 150ms (panel expand)
- Slow: 200ms (page transition)

**Why So Fast?**
- Developer tools need efficiency
- Reduce waiting feeling
- Avoid sluggishness

#### Animations to Avoid

- Rotating spinners (use solid progress bar instead)
- Bounce effects
- Complex 3D effects
- Long animations (> 300ms)

### Feedback Mechanisms

**Instant, Clear**

- Button click: Immediate background change
- Input focus: Immediate border highlight
- Operation success: Toast notification (1.5s auto-dismiss)
- Operation failure: Red border + error text

---

## Layout Principles

### Spacing Strategy

**4px Grid System**

- All spacing is multiples of 4
- Compact but not crowded
- Suitable for high information density scenarios

**Common Spacing:**

- Small spacing: 4px, 8px (related elements)
- Medium spacing: 12px, 16px (inside components)
- Large spacing: 24px, 32px (between modules)

### Compact Layout

**Information Density First**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ricocc/rico-skills](https://github.com/ricocc/rico-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
