---
trigger: always_on
description: Blazor WebAssembly demo site for the libphonenumber-csharp library, deployed to GitHub Pages.
---

# CLAUDE.md — PhoneNumbers.Demo

Blazor WebAssembly demo site for the libphonenumber-csharp library, deployed to GitHub Pages.

## Build and run

```bash
dotnet build csharp/PhoneNumbers.Demo
dotnet run --project csharp/PhoneNumbers.Demo
```

Publish for GitHub Pages (static WASM output):

```bash
dotnet publish csharp/PhoneNumbers.Demo -c Release
```

The demo references the main `PhoneNumbers` project directly — changes to the library are reflected immediately on rebuild.

## Styling conventions

### No inline styles

Never use `style="..."` attributes in Razor (`.razor`) or HTML (`.html`) files. All visual styling must live in CSS files:

- Move styles to the relevant per-component CSS file under `wwwroot/css/`.
- If the style is a one-off variant, add a BEM modifier class and define it in CSS.
- The only exception is Blazor's own auto-generated inline styles (e.g. from `@bind` or Blazor framework internals) — do not suppress those.

### CSS custom properties

All themable values are defined as CSS custom properties in `wwwroot/css/app.css` on `:root`. Use existing variables (`--primary`, `--text`, `--border`, `--radius`, etc.) rather than hard-coding colours or sizes.

### BEM naming

All component styles must use BEM (`block__element--modifier`).

- **Block**: the component name in kebab-case — `hero`, `card`, `nav-link`, `result-grid`
- **Element**: a child owned by the block, joined with `__` — `hero__content`, `card__title`, `result-grid__item`
- **Modifier**: a state or variant, joined with `--` — `badge--success`, `btn--primary`, `form-input--lg`

Rules:

- Each component owns one block. Don't reuse another component's block prefix.
- No bare element selectors (`h2`, `p`, `.container`) in component stylesheets — scope everything under the block.
- Modifiers stack on top of the base element class (`class="badge badge--success"`), they don't replace it.
- Avoid descendant selectors across blocks. If `.foo .bar` is tempting, `.bar` probably belongs to `.foo` as `.foo__bar`.
- Don't nest deeper than block > element > modifier in CSS — keep BEM flat and greppable.

### Per-component CSS files

Separate CSS into per-component files where appropriate:

- `wwwroot/css/app.css` — global reset, CSS custom properties, base typography, utility classes.
- Create dedicated CSS files for distinct components (e.g. `sidebar.css`, `card.css`, `hero.css`, `form.css`) when they grow beyond ~50 lines or are reused across multiple pages.
- Import component CSS files from `index.html` or via Blazor CSS isolation (`Component.razor.css`) where it makes sense.
- Shared layout styles (sidebar, main content scaffold) stay in a layout-level file.
- Page-specific one-off styles can live in a Blazor-scoped CSS file (`Pages/PageName.razor.css`).

### Theming

The app currently uses a light theme only. If dark mode is added:

- Define themable values as CSS custom properties with light defaults.
- Override under `[data-theme="dark"]`.
- All foreground/background pairs must meet WCAG AA contrast in both themes (see Accessibility below).

## Accessibility (WCAG AA compliance)

All changes must meet WCAG 2.1 Level AA. This is a hard requirement, not a nice-to-have.

### Colour and contrast

- All text/background pairs must meet 4.5:1 contrast ratio for normal text, 3:1 for large text (18px+ or 14px+ bold) and UI components/graphical objects.
- Verify contrast in both light theme and any future dark theme.
- Don't rely on colour alone to convey meaning — pair with an icon, text label, or shape (e.g. valid/invalid uses both colour and badge text).

### Semantics and structure

- Use semantic HTML: `<nav>`, `<main>`, `<section>`, `<h1>`–`<h6>` in correct hierarchy, `<button>` for actions, `<a>` for navigation.
- All interactive elements must be keyboard-accessible (focusable, operable via Enter/Space, visible focus indicator).
- All form inputs must have associated `<label>` elements (or `aria-label`/`aria-labelledby`).
- Images and icons that convey meaning need `alt` text or `aria-label`. Decorative icons use `aria-hidden="true"`.

### Motion and interaction

- Respect `prefers-reduced-motion`: wrap non-essential animations/transitions in `@media (prefers-reduced-motion: no-preference)`.
- Focus states must be visible — never `outline: none` without a replacement indicator.
- Touch targets must be at least 44x44px on mobile.

### Testing accessibility

- Run axe or Lighthouse accessibility audit before considering a UI change complete.
- Verify keyboard navigation through all interactive flows.
- Check screen reader announcements for dynamic content changes (e.g. parse results appearing).

## Component patterns

### Razor pages

Pages live in `Pages/` and use `@page "/route"`. Each page is self-contained with its `@code` block — no separate code-behind files unless complexity demands it.

### Shared state

Use `PhoneNumberUtil.GetInstance()` for the singleton — don't construct new instances. The instance is expensive and thread-safe.

### Error handling

Wrap `PhoneNumberUtil` calls that can throw (e.g. `Parse`) in try/catch for `NumberParseException`. Display user-friendly error messages inline using `.error-message`.

### Responsive design


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twcclegg/libphonenumber-csharp](https://github.com/twcclegg/libphonenumber-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
