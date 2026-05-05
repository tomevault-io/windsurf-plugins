---
trigger: always_on
description: You are an expert in Web Development Typescript, Next.js App Router, Tailwind CSS, Supabase
---


# Project Context

You are an expert in Web Development Typescript, Next.js App Router, Tailwind CSS, Supabase

Code Style and Structure:

- Write concise, technical Typescript code with accurate examples.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  -Structure files: exported component, subcomponents, helpers, static content

Naming Conventions:

- Use kebab-case for directories.
- Use camelCase for variables and functions.
- Use PascalCase for components.
- File names for components should be in PascalCase. Rest of the files in kebab-case.
- Prefix component names with their type (e.g. ButtonAccount.jsx and ButtonSignin.jsx, CardAnalyticsMain.jsx and CardAnalyticsData.jsx, etc.)

Syntax and Formatting:

- Use the "function" keyword for pure functions.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Use TSX.

UI and Styling:

- Use HextaUI and Tailwind CSS for components and styling.
- Implement responsive design with Tailwind CSS; use a mobile-first approach.
- You must use base components from "@/components/ui/"

Performance Optimization:

- Minimize 'use client', 'useState', and 'useEffect'; favor React Server Components (RSC).
- Wrap client components in Suspense with fallback.
- Use dynamic loading for non-critical components.
- Optimize images: use WebP format, include size data, implement lazy loading.

Key Conventions:

- Optimize Web Vitals (LCP, CLS, FID).
- Limit 'use client':
- Favor server components and Next.js SSR.
- Use only for Web API access in small components.
- Avoid for data fetching or state management.
- If absolutely necessary, you can use 'swr' library for client-side data fetching.
- When using client-side hooks (useState and useEffect) in a component that's being treated as a Server Component by Next.js, always add the "use client" directive at the top of the file.
- Follow Next.js docs for Data Fetching, Rendering, and Routing.

Design:

- For all designs I ask you to make, have them be beautiful, not cookie cutter. Make webpages that are fully featured and worthy for production.
- Do not install other packages for UI themes, icons, etc unless absolutely necessary or I request them.

Ultracite enforces strict type safety, accessibility standards, and consistent code quality for JavaScript/TypeScript projects using Biome's lightning-fast formatter and linter.

## Key Principles

- Zero configuration required
- Subsecond performance
- Maximum type safety
- AI-friendly code generation

## Before Writing Code

1. Analyze existing patterns in the codebase
2. Consider edge cases and error scenarios
3. Follow the rules below strictly
4. Validate accessibility requirements

## Rules

### Accessibility (a11y)

- Don't use `accessKey` attribute on any HTML element.
- Don't set `aria-hidden="true"` on focusable elements.
- Don't add ARIA roles, states, and properties to elements that don't support them.
- Don't use distracting elements like `<marquee>` or `<blink>`.
- Only use the `scope` prop on `<th>` elements.
- Don't assign non-interactive ARIA roles to interactive HTML elements.
- Make sure label elements have text content and are associated with an input.
- Don't assign interactive ARIA roles to non-interactive HTML elements.
- Don't assign `tabIndex` to non-interactive HTML elements.
- Don't use positive integers for `tabIndex` property.
- Don't include "image", "picture", or "photo" in img alt prop.
- Don't use explicit role property that's the same as the implicit/default role.
- Make static elements with click handlers use a valid role attribute.
- Always include a `title` element for SVG elements.
- Give all elements requiring alt text meaningful information for screen readers.
- Make sure anchors have content that's accessible to screen readers.
- Assign `tabIndex` to non-interactive HTML elements with `aria-activedescendant`.
- Include all required ARIA attributes for elements with ARIA roles.
- Make sure ARIA properties are valid for the element's supported roles.
- Always include a `type` attribute for button elements.
- Make elements with interactive roles and handlers focusable.
- Give heading elements content that's accessible to screen readers (not hidden with `aria-hidden`).
- Always include a `lang` attribute on the html element.
- Always include a `title` attribute for iframe elements.
- Accompany `onClick` with at least one of: `onKeyUp`, `onKeyDown`, or `onKeyPress`.
- Accompany `onMouseOver`/`onMouseOut` with `onFocus`/`onBlur`.
- Include caption tracks for audio and video elements.
- Use semantic elements instead of role attributes in JSX.
- Make sure all anchors are valid and navigable.
- Ensure all ARIA properties (`aria-*`) are valid.
- Use valid, non-abstract ARIA roles for elements with ARIA roles.
- Use valid ARIA state and property values.
- Use valid values for the `autocomplete` attribute on input elements.
- Use correct ISO language/country codes for the `lang` attribute.

### Code Complexity and Quality

- Don't use consecutive spaces in regular expression literals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preetsuthar17/ikiform](https://github.com/preetsuthar17/ikiform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
