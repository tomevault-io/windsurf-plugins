---
trigger: always_on
description: Writing CSS, whether inside .css files or in the `{% stylesheet %}…{% endstylesheet %}` or `{% style %}…{% endstyle %}` or in <style> </style> tags
---


# CSS Standards

## Specificity Rules

- **Never** use IDs as selectors
- **Avoid** using elements as selectors
- **Avoid** using `!important` at all costs - if you must use it, comment why in the code
- Use a `0 1 0` specificity wherever possible, meaning a single `.class` selector.
- In cases where you must use higher specificity due to a parent/child relationship, try to keep the specificity to a maximum of `0 4 0`
  - Note that this can sometimes be impossible due to the `0 1 0` specificity of pseudo-classes like `:hover`. There may be situations where `.parent:hover .child` is the only way to achieve the desired effect.
- **Avoid** complex selectors. A selector should be easy to understand at a glance. Don't over do it with pseudo selectors (:has, :where, :nth-child, etc).

See [MDN](mdc:https:/developer.mozilla.org/en-US/docs/Web/CSS/Specificity) for more a comprehensive list of specificity rules.

## CSS Variables

CSS variables, a.k.a. custom properties, are a powerful tool for reducing redundancy and making it easier to update values across a component.

- If you need to hardcode a value, set it to a variable and use that variable in the declaration. Example: a touch target size. `--touch-target-size: 44px;`
- **Never** hardcode colors, always use the color schemes

### Global Variables

Global variables should be scoped to the `:root` selector in `snippets/theme-styles-variables.liquid`.

**Example of global variables**

```css
/* in snippets/theme-styles-variables.liquid */
:root {
    --page-width: 1400px;
     --font-body--family: {{ settings.type_body_font.family }}, {{ settings.type_body_font.fallback_families }}; /* Referencing a theme setting */
     --font-{{ preset_name_dash }}--family: {{ settings[preset_font] | prepend: 'var(--font-' | append: '--family)' }}; /* Using Liquid to set a variable */
}
```

### Scoped Variables

Be sure to scope your CSS variables to the component they are being used in, if they are not meant to be global. Scoped variables can reference global variables.

**Example of scoped variables**

```css
/* in assets/facets.css */
.facets {
  --drawer-padding: var(--padding-md); /* Referencing a global variable */
  --facets-upper-z-index: 3;
  --facets-open-z-index: 4;

  --facets-clear-shadow: 0px -4px 14px 0px rgb(var(--color-foreground-rgb) / var(--opacity-10)); /* Referencing a Color Scheme variable */
}
```

### Namespace Your CSS Variables

Namespace your variables to avoid collisions unless you explicitly want them to bleed through to other components.

✅ Do this:

```css
.component {
  --component-padding: ...;
  --component-aspect-ratio: ...;
}
```

❌ Don't do this:

```css
.component {
  --padding: ...;
  --aspect-ratio: ...;
}
```

### Semantic Color Variables

Use semantic naming for better maintainability:

```css
:root {
  /* Base colors */
  --color-primary: {{ settings.colors_accent_1 }};
  --color-secondary: {{ settings.colors_accent_2 }};

  /* Semantic colors */
  --color-text-primary: rgb(var(--color-foreground));
  --color-text-secondary: rgb(var(--color-foreground) / 0.75);
  --color-text-disabled: rgb(var(--color-foreground) / 0.38);

  /* Interactive states */
  --color-interactive-default: rgb(var(--color-accent));
  /* color-mix isn't supported in earlier version of iOS <16.2 so limit its usage to progressive enhancement */
  --color-interactive-hover: color-mix(in srgb, rgb(var(--color-accent)) 90%, black);
  --color-interactive-pressed: color-mix(in srgb, rgb(var(--color-accent)) 80%, black);
  --color-interactive-disabled: rgb(var(--color-accent) / 0.38);
}
```

### Design Token System

Establish consistent spacing and typography scales:

```css
:root {
  /* Spacing scale */
  --space-3xs: 0.25rem; /* 4px */
  --space-2xs: 0.5rem; /* 8px */
  --space-xs: 0.75rem; /* 12px */
  --space-sm: 1rem; /* 16px */
  --space-md: 1.5rem; /* 24px */
  --space-lg: 2rem; /* 32px */
  --space-xl: 3rem; /* 48px */
  --space-2xl: 4rem; /* 64px */
  --space-3xl: 6rem; /* 96px */

  /* Typography scale */
  --font-size-xs: 0.75rem; /* 12px */
  --font-size-sm: 0.875rem; /* 14px */
  --font-size-base: 1rem; /* 16px */
  --font-size-lg: 1.125rem; /* 18px */
  --font-size-xl: 1.25rem; /* 20px */
  --font-size-2xl: 1.5rem; /* 24px */
  --font-size-3xl: 1.875rem; /* 30px */
}
```

## Scoping CSS to Instances of Sections and Blocks

Reset CSS variable values inline on a `style` attribute with a section/block settings. This has a couple benefits:

- Less CSS in Liquid which allows us to use the `{% stylesheet %}` tag for all CSS.
- Reduces redundancy in CSS selectors and number of selectors in the HTML, i.e. `.selector--{{ block.id }}` pattern.

✅ Do this:

```html
<section
  style="
    --background-color: {{ settings.background_color }};
    --padding: {{ settings.padding }}px;
  "
>
  ...
</section>

<button style="--button-color: {{ settings.button_color }};">...</button>
```

❌ Don't do this:

```html
{% style %} .selector--{{ block.id }} { --button-color: {{ settings.button_color }}; } {% endstyle %}

<button class="selector--{{ block.id }}">...</button>
```

### Redundancy

Use variables to reduce property assignment redundancy.

```css
/* Do this */
.button {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EcomExperts-io/Base](https://github.com/EcomExperts-io/Base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
