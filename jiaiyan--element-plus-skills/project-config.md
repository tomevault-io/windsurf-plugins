---
trigger: always_on
description: Element Plus Skills Library - A comprehensive skill library for AI agents to understand and utilize Element Plus UI components. Invoke when user needs to work with Element Plus components, theming, i18n, dark mode, or design specifications.
---


# Element Plus Skills Library

A comprehensive skill library for Element Plus UI framework, designed for AI agents to understand and utilize Element Plus components effectively.

## When to Invoke

Invoke this skill when:
- User needs to implement any Element Plus component
- User asks about Element Plus configuration or setup
- User wants to customize themes or use dark mode
- User needs internationalization (i18n) support
- User asks about design specifications (colors, borders, typography)
- User encounters issues with Element Plus components

## Skill Library Overview

This library contains **88 skills** organized into the following categories:

| Category | Count | Description | Path Pattern |
|----------|-------|-------------|--------------|
| Component Skills | 77 | Individual component documentation | `./components/el-{name}/SKILL.md` |
| Design Specifications | 5 | Border, Color, Layout, Typography, Overview | `./element-plus-design-{name}/SKILL.md` |
| Foundation Skills | 6 | Quickstart, Theming, i18n, Dark Mode, SSR, Components | `./element-plus-{name}/SKILL.md` |

## How to Locate Skills

### 1. Component Skills (77 skills)

All component skills follow the naming convention `el-{component-name}` and are located in the `components/` directory.

**Path Pattern:**
```
./components/el-{component-name}/SKILL.md
```

**Examples:**
- Button: `./components/el-button/SKILL.md`
- Form: `./components/el-form/SKILL.md`
- Table: `./components/el-table/SKILL.md`
- Dialog: `./components/el-dialog/SKILL.md`

### 2. Design Specification Skills (5 skills)

Design skills provide guidance on Element Plus design system.

| Skill Name | Path | Description |
|------------|------|-------------|
| Border | `./element-plus-design-border/SKILL.md` | Border styles, radius, shadows |
| Color | `./element-plus-design-color/SKILL.md` | Color palette and semantics |
| Layout | `./element-plus-design-layout/SKILL.md` | 24-column grid system |
| Typography | `./element-plus-design-typography/SKILL.md` | Font conventions |
| Overview | `./element-plus-design-overview/SKILL.md` | Design system overview |

### 3. Foundation Skills (6 skills)

Foundation skills cover core setup and configuration.

| Skill Name | Path | Description |
|------------|------|-------------|
| Quickstart | `./element-plus-quickstart/SKILL.md` | Installation and setup |
| Theming | `./element-plus-theming/SKILL.md` | Theme customization |
| i18n | `./element-plus-i18n/SKILL.md` | Internationalization |
| Dark Mode | `./element-plus-dark-mode/SKILL.md` | Dark mode implementation |
| SSR | `./element-plus-ssr/SKILL.md` | Server-side rendering |
| Components | `./element-plus-components/SKILL.md` | Component overview index |

## Skill Invocation Guide

### Step 1: Identify User Intent

Analyze the user's request to determine which skill category is needed:

| User Request Pattern | Skill Category | Example Skill |
|---------------------|----------------|---------------|
| "Create a button/form/table..." | Component | `el-button`, `el-form`, `el-table` |
| "How to set up Element Plus" | Foundation | `element-plus-quickstart` |
| "Customize theme colors" | Foundation | `element-plus-theming` |
| "Add multi-language support" | Foundation | `element-plus-i18n` |
| "Implement dark mode" | Foundation | `element-plus-dark-mode` |
| "What colors are available?" | Design | `element-plus-design-color` |
| "How does the grid work?" | Design | `element-plus-design-layout` |

### Step 2: Locate the Skill File

Use the path patterns above to locate the appropriate skill file:

```markdown
# For component skills
./components/el-{component-name}/SKILL.md

# For design skills
./element-plus-design-{name}/SKILL.md

# For foundation skills
./element-plus-{name}/SKILL.md
```

### Step 3: Read and Apply Skill Content

Each skill file contains:
- **When to Invoke**: Specific conditions for using the skill
- **Features**: Component capabilities and options
- **API Reference**: Attributes, events, slots, exposes
- **Usage Examples**: Code snippets for common patterns
- **Best Practices**: Recommended implementation guidelines

## Component Skill Index

### Basic Components (14)

| Component | Skill Path | Description |
|-----------|------------|-------------|
| Affix | `./components/el-affix/SKILL.md` | Sticky positioning |
| Alert | `./components/el-alert/SKILL.md` | Alert messages |
| Anchor | `./components/el-anchor/SKILL.md` | Anchor navigation |
| Avatar | `./components/el-avatar/SKILL.md` | User avatars |
| Backtop | `./components/el-backtop/SKILL.md` | Back to top button |
| Badge | `./components/el-badge/SKILL.md` | Badges and marks |
| Breadcrumb | `./components/el-breadcrumb/SKILL.md` | Breadcrumb navigation |
| Button | `./components/el-button/SKILL.md` | Buttons |
| Card | `./components/el-card/SKILL.md` | Card containers |
| Carousel | `./components/el-carousel/SKILL.md` | Image carousels |
| Collapse | `./components/el-collapse/SKILL.md` | Collapsible panels |
| Divider | `./components/el-divider/SKILL.md` | Dividing lines |
| Icon | `./components/el-icon/SKILL.md` | SVG icons |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jiaiyan/element-plus-skills](https://github.com/jiaiyan/element-plus-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
