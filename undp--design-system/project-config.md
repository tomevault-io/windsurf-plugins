---
trigger: always_on
description: These instructions guide AI code generation (Copilot / Chat) to produce consistent, high‑quality code for this UNDP Design System. The tech stack centers on React (v19), Storybook 10 (Webpack 5 builder), SCSS, and a hybrid of authored JS utilities plus asset pipeline.
---

# GitHub Copilot Repository Instructions

These instructions guide AI code generation (Copilot / Chat) to produce consistent, high‑quality code for this UNDP Design System. The tech stack centers on React (v19), Storybook 10 (Webpack 5 builder), SCSS, and a hybrid of authored JS utilities plus asset pipeline.

---
## Core Principles
1. Accessibility first (semantic HTML, aria attributes, keyboard navigation, color contrast).
2. Deterministic builds (avoid adding new runtime deps casually; prefer vanilla / existing utilities).
3. Storybook stories are documentation assets: every component must export a default with `title`, `argTypes`, `parameters`, and at least one well‑named story.
4. Keep bundle surface minimal: tree-shake friendly imports, no gratuitous re-exports, avoid large utility libraries unless already present.
5. Follow existing styling & naming conventions (`ComponentName`, SCSS partials, BEM‑like class structure already in repository assets).
6. Avoid mutating global state unless following existing patterns (`window.UNDP`, language/dir, initialization scripts, component Initializer).

---
## React Component Guidelines
- Use function components.
- Prefer explicit prop objects + JSDoc for any new props.
- Keep side effects in `useEffect` with correct dependency arrays.
- Do **not** introduce state management libraries—prefer local state or simple context if absolutely necessary (ask before adding context providers).
- Derive, don’t duplicate: compute values from props rather than storing duplicates in state.
- Accessibility:
  - Provide `aria-label`, `aria-current`, roles, and heading hierarchy where applicable.
  - Use semantic elements (`nav`, `ul`, `button`, `section`).
  - Focus management where modals / menus are involved.

---
## Storybook Conventions
### File & Export Pattern
- Story files live alongside component folders: `ComponentName.stories.js(x)`.
- Default export object must include:
  - `title`: Follow existing taxonomy (e.g. `Components/Navigation components/Breadcrumbs`).
  - `argTypes`: Only expose curated controls. Always provide `description` and (where helpful) `name` for UI label clarity.
  - `parameters.docs.page`: Use inline JSX doc layout pattern with `<Canvas><Story/></Canvas>` as seen in `Breadcrumbs.stories.js`.

### Args & Controls
- Use `argTypes.colorTheme` with:
  ```js
  colorTheme: {
    name: 'Color theme',
    control: { type: 'inline-radio' },
    options: ['light', 'dark'],
    description: 'Color Theme',
  }
  ```
- Prefer `inline-radio` for small enumerations, `select` for >4 options, and `boolean` controls only when semantically meaningful.
- Avoid exposing internal implementation props (only document consumer-facing API).

### Background / Theme Handling
- Background color is driven **only** by the `colorTheme` arg through the custom decorator `setColorThemeBackgroundFromArgs` in `preview.js`.
- Do **not** reintroduce the Storybook Backgrounds toolbar or global UPDATE_GLOBALS emission for theme switching.
- If a story must force dark mode, set:
  ```js
  MyStory.args = { colorTheme: 'dark' };
  ```

### Locale / Accent Globals
- Respect existing `globalTypes.locale` and `globalTypes.accent`.
- Do not add new global toolbar items without justification.
- For locale-dependent stories, access locale from `context.globals.locale` and pass through to helpers (see Breadcrumb implementation pattern).

### Docs & MDX
- Current approach: in-JSX docs via `parameters.docs.page` — continue this pattern.
- When adding usage sections, list script & CSS dependencies referencing CDN pattern used elsewhere.

### Story Naming
- Primary example story uses component name as `Story.storyName` if needed for spacing. Avoid duplicate naming (Storybook handles export variable names by default).
- Use PascalCase export variable names.

---
## SCSS & Styling
- Reuse existing tokens/utilities from imported base SCSS (`base-minimal.scss`).
- Keep new component SCSS minimal and colocated.
- Use nesting responsibly; avoid >3 levels deep.
- Generate only necessary classes; rely on existing utility classes where possible.

---
## JavaScript Utilities Patterns
For non-React behaviors (e.g., `fitTitle`, component initializers):
- Use pure functions where feasible.
- Avoid polluting global scope; attach only to `window.UNDP` if following an existing namespace pattern.
- When measuring layout (like text width), batch DOM reads then writes (if adding new logic) to reduce layout thrash.
- Use `window.getComputedStyle` to clone typography when measuring (example: pseudo measurement in `fitTitle`).

---
## Initialization Script (`init.js`)
- Cases in the large switch are alphabetically sorted—maintain that order.
- When adding a new component type:
  1. Add a new `case 'component-name':` block alphabetically.
  2. Guard with `if (typeof someFunction === 'function')`.
  3. Set `data-initialized="true"` after successful init.
  4. Provide a clear `console.warn` fallback message.

---
## Performance & Bundle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [undp/design-system](https://github.com/undp/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
