---
trigger: always_on
description: You are an expert full-stack accessibility developer. MagentaA11y V2 is an accessibility acceptance criteria documentation platform built with React and TypeScript. The project provides comprehensive accessibility guidelines, testing procedures, and best practices for web and native applications.
---

# AI Agent Instructions for MagentaA11y V2

## Project Overview
You are an expert full-stack accessibility developer. MagentaA11y V2 is an accessibility acceptance criteria documentation platform built with React and TypeScript. The project provides comprehensive accessibility guidelines, testing procedures, and best practices for web and native applications.

## Core Principles
1. **Accessibility First**: All code and documentation must follow WCAG 2.2 A & AA guidelines and accessibility best practices
2. **Semantic HTML**: Always use proper semantic markup
3. **ARIA Standards**: Follow WAI-ARIA 1.2 authoring practices correctly
4. **Testing Focus**: Maintain comprehensive testing documentation
5. **AI outputs**:Be concise in your outputs and not overly verbose.

## Code Style & Conventions

### TypeScript & React
- Use functional components with TypeScript
- Prefer named exports over default exports
- Use proper TypeScript types (avoid `any`)
- Follow React hooks best practices
- Keep components focused and modular

### File Structure
- Components go in `/src/components/` with kebab-case folder names
- Each component should have its own folder with `.tsx`, `.scss`, and `.test.tsx` files
- Shared types go in `/src/shared/types/`
- Utilities go in `/src/utils/`

### Markdown Content Components (Refactored)
The monolithic `markdown-content.tsx` has been refactored into a thin orchestrator that delegates rendering to individual element components under:
`src/components/content-display/markdown-content/elements/`

**Element components:**
- `markdown-button/` — interactive buttons with function mapping
- `markdown-div/` — divs with optional event handling
- `markdown-image/` — images with asset URL resolution
- `markdown-input/` — form inputs with function mapping and aria-disabled support
- `markdown-link/` — links (internal anchors, external, routed, function-mapped)
- `markdown-list-item/` — list items with interactive card support
- `markdown-source/` — `<source>` elements for video (includes Safari `#t=0.1` fix)
- `markdown-track/` — `<track>` elements for video captions (VTT files)
- `markdown-video/` — `<video>` elements with poster and preload handling

**Shared files in `elements/`:**
- `types.ts` — base interfaces (`BaseMarkdownElementProps`, `FunctionMappedProps`, `AriaDisabledProps`)
- `utils.ts` — shared helpers (`resolveAssetUrl`, `invokeMappedFunction`, `isAriaDisabled`, `wrapWithAriaDisabledCheck`)

**Pattern for adding a new element component:**
1. Create a folder under `elements/` (e.g., `markdown-foo/`)
2. Create `markdown-foo.types.ts` — export an interface extending `BaseMarkdownElementProps`
3. Create `markdown-foo.tsx` — use helpers from `../utils` as needed
4. Register in `markdown-content.tsx` components map: `foo: (props) => <MarkdownFoo {...props} assetBasePath={assetBasePath} />`

**Interactive markdown functions:**
`src/utils/markdownFunctions.ts` holds the function map for elements using `data-fn` and `data-event` attributes.

### Styling
- Use SCSS modules
- Follow BEM naming convention where applicable
- Maintain theme tokens in `/src/styles/`
- Ensure responsive design and dark mode support
- All interactive elements must have visible focus indicators

### SCSS Partials for Markdown Examples (Refactored)
The monolithic `_md-styles.scss` has been split into individual partials under `src/styles/md-styles/`. The main `_md-styles.scss` is now an import manifest — do not add styles directly to it.

**Partials:**
- `_tables.scss` — table styles
- `_keyboard.scss` — keyboard shortcut display styles
- `_example-base.scss` — shared base styles for all examples
- `_example-carousel.scss` — carousel demo styles
- `_example-switch.scss` — toggle switch demo styles
- `_example-stepper.scss` — stepper demo styles
- `_example-forms.scss` — form element demo styles
- `_example-select.scss` — select/dropdown demo styles
- `_example-links.scss` — link demo styles
- `_example-alerts.scss` — alert/notification demo styles
- `_example-expander.scss` — expandable/accordion demo styles
- `_example-password.scss` — password input demo styles
- `_example-tidbit.scss` — tidbit component demo styles
- `_example-spa.scss` — SPA navigation demo styles
- `_cards.scss` — card layout styles
- `_stars.scss` — star rating demo styles
- `_animation.scss` — animation/motion demo styles
- `_bad-example.scss` — inaccessible "bad example" indicator styles
- `_sticky.scss` — sticky positioning styles

**Naming convention:** `_example-{component}.scss` for component-specific demo styles. Add new partials following this pattern and import them in `_md-styles.scss`.

### Documentation (Markdown)
- All documentation files go in `/public/content/documentation/`
- Follow the structure: `<section>/<category>/<filename>.md`
- Use the provided templates: `criteria` or `how-to-test`
- Use the script: `npm run create-md -- <filename> "<relative-path>" <template-type>`
- Templates are outputted by `src/scripts/createMarkdown.sh`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmobile/magentaA11y](https://github.com/tmobile/magentaA11y) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
