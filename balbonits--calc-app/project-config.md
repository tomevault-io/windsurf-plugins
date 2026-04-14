---
trigger: always_on
description: - In `globals.css`, use layered imports:
---

# Claude Code Context
#
# ---
# Tailwind CSS v4 Integration Guide
#

# Tailwind CSS v4 Integration Guide


# Tailwind CSS v4 Integration Guide

## Usage
- In `globals.css`, use layered imports:
	- `@import "tailwindcss/theme.css" layer(theme);`
	- `@import "tailwindcss/preflight.css" layer(base);`
	- `@import "tailwindcss/utilities.css" layer(utilities);`
- Define design tokens and theme variables in `@layer theme { :root { ... } }` in `globals.css` (e.g., `--color-gray-900: #171717;`).
- For dark mode, use `:global(.dark)` inside `@layer theme`.
- Use `@layer base { body { ... } }` for base element styles.
- In CSS modules (e.g., `Calculator.module.css`), use only local class or ID selectors (e.g., `.calculator`, `.buttonGrid`). Do not use `@theme`, `@layer`, or `:root`.
- Reference global variables in CSS modules: `background: var(--color-gray-900);`.
- Use Tailwind utility classes directly in TSX/JSX or global CSS. Avoid `@apply` in CSS modules unless your build system supports it.
- Ensure `globals.css` is imported in your Next.js layout (`app/layout.tsx`).


## Troubleshooting
- If you see "Unknown at rule @theme" or similar warnings in your IDE or lint output, you can safely ignore them for Tailwind v4 custom at-rules. These are not build errors if your app compiles and runs.
- For best developer experience, install the Tailwind CSS IntelliSense extension in VS Code and add a `.tailwind.json` file to resolve warnings.
- You may also configure your linter to ignore `unknownAtRules` for CSS files:
	- For Stylelint, add to your config:
		```json
		"rules": {
			"at-rule-no-unknown": [true, {
				"ignoreAtRules": ["theme", "layer", "tailwind", "apply"]
			}]
		}
		```
- Ensure your `postcss.config.js` includes `tailwindcss` and `autoprefixer` plugins.

## Example
```css
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);

@layer theme {
	:root {
		--color-gray-900: #171717;
		--color-gray-100: #f3f4f6;
		/* ...other tokens... */
	}
	:global(.dark) {
		--background: #0a0a0a;
		--foreground: #ededed;
	}
}

@layer base {
	body {
		background: var(--background);
		color: var(--foreground);
		font-family: var(--font-sans);
		transition: background-color 0.2s, color 0.2s;
	}
}

/* In Calculator.module.css: */
.calculator {
	background: var(--color-gray-900);
	color: var(--color-gray-100);
	/* ...other local styles... */
}
```

---

---
**Reminder:**
Always update both CLAUDE.md and TODO.md whenever you make changes or additions to project context or tasks. This keeps documentation and progress tracking accurate and up-to-date.

**Workflow:**
 Always work through the TODO list step-by-step, completing each item in order before moving to the next.

**Error Prevention Reminder:**
 Always ensure:
 - The "use client" directive is the very first line in client components.
 - All imports are placed immediately after the directive.
 - Component logic and return statements are wrapped inside the function definition.
 - No duplicate imports or misplaced braces.
 - No extra or missing braces in CSS files; validate CSS syntax after every style change.
 This helps prevent common syntax and runtime errors in Next.js/React projects.

This file provides context for Anthropic Claude Code.

---
## Project Guide Addition

# How to Add a New Project - Complete Guide

This guide walks you through adding a new project to the showcase system.

## 📂 Step 1: Prepare Project Assets (Auto-Processing)

### 🎯 Simple Image Setup
Follow the naming convention and drop images in the right folder!

```
📁 raw-images/[project-slug]/
├── 📸 01-desktop-homepage.png         # Main desktop view
├── 📱 02-mobile-login.jpg             # Mobile screenshot
├── ⭐ 03-feature-dashboard.png        # Feature demonstration
└── 🎯 04-tablet-settings.webp         # Tablet view
```

### 📝 Required Naming Convention
**Format**: `[number]-[category]-[description].[ext]`

**📋 Parts Explained**:
- **number**: Display order (01, 02, 03...) - controls screenshot sequence in showcase
- **category**: Device type (desktop, mobile, tablet, feature) - determines output dimensions
- **description**: What it shows (homepage, login, dashboard) - for human identification
- **ext**: File format (png, jpg) - input format you provide

**📐 Categories & Output Sizes**:
- `desktop` → 1200×800px (main views, homepage, dashboard)
- `mobile` → 375×667px (mobile app screenshots)
- `tablet` → 768×1024px (tablet/iPad views) 
- `feature` → 800×600px (feature demos, components)

**Examples**:
```
✅ Perfect:
01-desktop-homepage.png          (1st screenshot, desktop size, homepage)
02-mobile-user-profile.jpg       (2nd screenshot, mobile size, profile screen)
03-feature-search-filters.png    (3rd screenshot, feature size, search demo)
04-tablet-dashboard.png          (4th screenshot, tablet size, dashboard)
```

---
#### Calculator App Portfolio Integration (Contextual Guide)

For the Simple Calculator app, prepare screenshots in `raw-images/simple-calculator/` using the format `[number]-[category]-[description].[ext]`.

Recommended images:
- `01-desktop-homepage.png` (main calculator view on desktop)
- `02-mobile-calculator.jpg` (calculator on mobile)
- `03-feature-error-message.png` (error handling example)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/balbonits) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
