---
trigger: always_on
description: I am an AI assistant helping with the development, testing, and maintenance of this project. I MUST ALWAYS follow these rules without exections!
---

# Agent Instructions

I am an AI assistant helping with the development, testing, and maintenance of this project. I MUST ALWAYS follow these rules without exections!

# General Instructions

- I MUST ONLY use Tailwind CSS v4 for styling - no custom CSS or other styling libraries.
- I MUST follow mobile-first design approach - all components MUST be fully responsive across all device sizes.
- I MUST NEVER start the development server as the user has it running in the background.
- I MUST ONLY make changes explicitly requested by the user. If I think additional changes would be beneficial, I MUST ask for confirmation first.
- I MUST ensure all code is production-ready, properly tested, and follows project conventions.
- I MUST accurately implement designs from Figma with pixel-perfect precision and proper responsive behavior.
- I MUST use Perplexity search proactively in these specific situations:
  - Before implementing any feature to research current best practices
  - When working with external APIs or libraries to verify syntax and usage patterns
  - When troubleshooting errors or unexpected behavior
  - When evaluating architectural decisions to ensure industry standards
  - To research security considerations for sensitive operations
  - When suggesting improvements to existing code or documentation
  - To verify compatibility with different browsers or platforms

# MANDATORY SELF-CHECK BEFORE EVERY RESPONSE

Before submitting any response that involves code, I MUST verify:

1. That I am ONLY using Tailwind CSS v4 for styling (no custom CSS)
2. That all code follows mobile-first responsive design principles
3. That I'm not suggesting launching the development server
4. That I'm only making changes explicitly requested by the user
5. For Supabase code: I am ONLY using approved patterns (detailed in Supabase section)
6. That UI components follow accessibility best practices
7. That my code follows the project's file organization structure
8. That TypeScript types are properly used with no 'any' types
9. That I've kept components under 200 lines with single responsibilities
10. That I ALWAYS follow DRY (Don't Repeat Yourself) principles in all code
11. That I'm using the @theme directive properly for design tokens instead of configuration files
12. That I've created separate pages with distinct URLs for each major phase of a workflow (e.g., separate pages for upload forms and result displays)
13. That I've implemented immediate visual feedback for all user interactions, especially for processes that take time (loading states, progress indicators, success/error messages)
14. That I've accurately implemented Figma designs with proper design tokens, spacing, typography, and component structure

# Tailwind CSS v4 Implementation

## Setup and Configuration

- I MUST understand that Tailwind CSS v4 uses the @theme directive in CSS to define theme variables, rather than configuration in JavaScript/TypeScript files.
- I MUST ensure that project has the correct dependencies:
  - tailwindcss (the core package)
  - @tailwindcss/postcss (for PostCSS integration)
  - postcss
- I MUST check that the CSS file imports Tailwind correctly using `@import "tailwindcss";` at the top of the main CSS file.
- I MUST confirm the postcss.config.mjs file includes `"@tailwindcss/postcss": {}` in its plugins.
- I MUST NEVER modify the compiled CSS output directly.

## Theme Variables and the @theme Directive

- I MUST define all design tokens using the @theme directive in the CSS file:
  ```css
  @import 'tailwindcss';
  @theme {
    --color-brand-primary: #3b82f6;
    --color-brand-secondary: #1e40af;
    /* Add all other design tokens here */
  }
  ```
- I MUST understand that theme variables are special CSS variables with these properties:
  - They MUST be defined at the top level (not nested inside other selectors)
  - They generate corresponding utility classes automatically (e.g., `--color-mint-500` creates `bg-mint-500`, `text-mint-500`, etc.)
  - They are also available as regular CSS variables for use in arbitrary values
- I MUST follow these variable naming conventions to map to the correct utility classes:
  - `--color-*`: Creates text, background, border, and other color utilities
  - `--font-*`: Creates font-family utilities
  - `--breakpoint-*`: Creates responsive variants (e.g., `--breakpoint-3xl: 120rem;` creates the `3xl:` breakpoint)
  - And other namespaces as defined in the Tailwind CSS documentation

## @theme vs. :root Variables

- I MUST use @theme for variables that should generate utility classes (design tokens)
- I SHOULD use regular :root CSS variables for values that don't need corresponding utility classes
- I MUST understand that @theme variables automatically create CSS variables that can be used with var() syntax

# User Experience and Accessibility (UX & a11y)

## Responsive Design & UI

- I MUST build responsive interfaces that work across modern browsers and devices.
- I MUST use Tailwind CSS v4 for consistent spacing and responsive layouts.
- I MUST ensure interactive elements are easily tappable on mobile (minimum 44x44px touch targets) and keyboard-navigable on desktop.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samyhajar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
