---
trigger: always_on
description: EldoraUI is a modern UI component library built with React, TypeScript, and Tailwind CSS. This document defines the coding standards, component patterns, and development workflows for maintaining high-quality, accessible, and performant components.
---

# EldoraUI Development Standards

EldoraUI is a modern UI component library built with React, TypeScript, and Tailwind CSS. This document defines the coding standards, component patterns, and development workflows for maintaining high-quality, accessible, and performant components.

## Project Overview
- **Framework**: Next.js 15 with React 19
- **Language**: TypeScript with strict type checking
- **Styling**: Tailwind CSS with custom animations
- **Component System**: shadcn/ui compatible registry
- **Documentation**: MDX-based with live examples
- **Package Manager**: pnpm with workspace support

## Key Principles
- **Component-First Architecture**: Every UI element is a reusable, composable component
- **Type Safety**: Strict TypeScript with comprehensive prop interfaces
- **Accessibility First**: WCAG 2.1 AA compliance for all components
- **Performance Optimized**: Minimal bundle size with tree-shaking support
- **Developer Experience**: Excellent IntelliSense and documentation

## Before Writing Code
1. **Analyze existing component patterns** in `/apps/www/registry/eldoraui/`
2. **Review component demos** in `/apps/www/registry/example/`
3. **Check documentation structure** in `/apps/www/content/docs/components/`
4. **Follow the registry system** defined in `/apps/www/registry/`
5. **Validate accessibility requirements** and test with screen readers

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
- Don't use the `arguments` object.
- Don't use primitive type aliases or misleading types.
- Don't use the comma operator.
- Don't use empty type parameters in type aliases and interfaces.
- Don't write functions that exceed a given Cognitive Complexity score.
- Don't nest describe() blocks too deeply in test files.
- Don't use unnecessary boolean casts.
- Don't use unnecessary callbacks with flatMap.
- Use for...of statements instead of Array.forEach.
- Don't create classes that only have static members (like a static namespace).
- Don't use this and super in static contexts.
- Don't use unnecessary catch clauses.
- Don't use unnecessary constructors.
- Don't use unnecessary continue statements.
- Don't export empty modules that don't change anything.
- Don't use unnecessary escape sequences in regular expression literals.
- Don't use unnecessary fragments.
- Don't use unnecessary labels.
- Don't use unnecessary nested block statements.
- Don't rename imports, exports, and destructured assignments to the same name.
- Don't use unnecessary string or template literal concatenation.
- Don't use String.raw in template literals when there are no escape sequences.
- Don't use useless case statements in switch statements.
- Don't use ternary operators when simpler alternatives exist.
- Don't use useless `this` aliasing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karthikmudunuri/eldoraui](https://github.com/karthikmudunuri/eldoraui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
