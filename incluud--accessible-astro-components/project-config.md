---
trigger: always_on
description: A comprehensive library of accessible, easy-to-use UI components for Astro websites, built with WCAG compliance and inclusive design principles.
---

# Accessible Astro Components

A comprehensive library of accessible, easy-to-use UI components for Astro websites, built with WCAG compliance and inclusive design principles.

## Project Overview

- **Type**: NPM package / component library
- **Published**: https://www.npmjs.com/package/accessible-astro-components
- **Documentation**: https://accessible-astro.incluud.dev/components/overview/
- **Repository**: https://github.com/incluud/accessible-astro-components

Check `package.json` for current version and dependencies.

### Key Characteristics

- **Zero dependencies**: Pure Astro components
- **NPM package**: Consumed by multiple projects (starter, dashboard, docs)
- **Shared styles**: Exports `index.css` with all component styles
- **TypeScript**: Full type definitions in `src/types/index.d.ts`
- **Modern CSS**: Uses logical properties, custom properties, `light-dark()` for theming

## Package Structure

This is an **NPM package** that exports components and styles:

```
src/
├── components/          # All UI components organized by feature
│   ├── accordion/       # Expandable content sections
│   ├── avatar/          # User avatars and avatar groups
│   ├── badge/           # Labels and status indicators
│   ├── forms/           # Complete form component suite
│   ├── tabs/            # Tabbed interface components
│   └── ... (see "Available Components" below)
├── styles/              # Shared CSS exported with the package
│   └── index.css        # Main styles file (imports all component styles)
└── types/
    └── index.d.ts       # TypeScript definitions for all components

index.js                 # Main export file - check here for full component list
```

See `index.js` for the complete, current list of exported components.

## Available Components

Check `index.js` for the current, complete list. Major component categories include:

### Navigation & Structure

- **Accordion** / **AccordionItem**: Expandable sections with keyboard support
- **Breadcrumbs** / **BreadcrumbsItem**: Navigation trail
- **Pagination**: Paginated content navigation
- **SkipLink**: Keyboard navigation aid
- **Tabs** suite: Tabbed interface with proper ARIA support

### Interactive Elements

- **Button**: Accessible button with variants and animations
- **Link**: Link component for internal/external navigation
- **Modal**: Accessible dialog with focus trap
- **DarkMode**: Theme toggle with system preference support

### Forms (Complete Suite)

- **Form**: Form wrapper with validation
- **Input**: Text inputs with validation states
- **Textarea**: Multi-line text input
- **Checkbox**: Accessible checkbox with proper labeling
- **Radio**: Radio buttons with proper grouping
- **Fieldset**: Form field grouping with legend

### Content Display

- **Card**: Flexible content container
- **Avatar** / **AvatarGroup**: User avatars with images, initials, or icons
- **Badge**: Labels and status indicators with animations
- **Notification**: Info and alert messages
- **Media**: Responsive image component
- **Video**: YouTube embed component
- **Heading**: Semantic heading with proper hierarchy

## Dev Environment Setup

1. **Install dependencies**:

   ```bash
   npm install
   ```

2. **Development workflow**:

   - This is a pure component library (no dev server)
   - Test components in consuming projects (starter, dashboard, docs)
   - Use symlinks for local testing (see below)

3. **Link to consuming projects**:

   ```bash
   # From the components directory
   npm link

   # Then in consuming project (starter/dashboard/docs)
   cd ../accessible-astro-starter
   npm link accessible-astro-components
   ```

4. **Format code**:
   ```bash
   npx prettier --write .
   ```

## Code Style Guidelines

### Component Architecture

- **Single Responsibility**: Each component has one clear purpose
- **Composition**: Components can be nested (e.g., Accordion + AccordionItem)
- **Props**: Use Astro props with TypeScript types
- **Styles**: Component styles in `src/styles/`, imported via `index.css`
- **No Dependencies**: Pure Astro components only

### TypeScript

- Type definitions in `src/types/index.d.ts`
- Export all component prop types
- Use Astro's `Astro.props` typing

### Formatting

- **Prettier** configured with:
  - `prettier-plugin-astro`
  - `prettier-plugin-css-order`
- Run: `npx prettier --write .`

### CSS Guidelines

- **Modern CSS**: Use logical properties (`inline-start`, `block-end`, etc.)
- **Custom Properties**: Use CSS variables for theming
- **Dark Mode**: Use `light-dark()` function for color theming
- **No Frameworks**: Pure CSS, no Tailwind or other frameworks
- **BEM-style**: Use clear class naming conventions
- **Performance**: Keep specificity low, avoid deep nesting

## Accessibility Requirements

All components follow **WCAG 2.2 AA** standards:

### Essential Practices

1. **Semantic HTML**: Use proper elements (`<button>`, `<nav>`, `<dialog>`, etc.)
2. **Keyboard Navigation**: All interactive components are keyboard accessible
3. **Focus Management**: Visible focus indicators, proper focus trapping (modals)
4. **ARIA**: Use ARIA attributes when necessary (roles, states, properties)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [incluud/accessible-astro-components](https://github.com/incluud/accessible-astro-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
