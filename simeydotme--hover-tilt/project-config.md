---
trigger: always_on
description: Important styling considerations for the HoverTilt web component
---


# Web Component Shadow DOM Styling Rules

When creating demos or documentation examples for the HoverTilt web component, follow these critical styling rules to ensure the component works correctly.

## Prefer CSS Variables Over data-is-active

The HoverTilt component exposes CSS variables (like `--hover-tilt-opacity`) to slotted content via `::slotted(*)`. This is the **preferred way** to make slotted content respond to the component's state.

### ✅ Recommended: Use CSS Variables

```css
/* ✅ BEST - Use the CSS variable directly */
.my-element {
  transform: translateZ(calc(var(--hover-tilt-opacity, 0) * 50px));
}
```

The `--hover-tilt-opacity` variable animates from 0 to 1 when active, so you don't need to check `data-is-active` at all.

### When You Need data-is-active

Only use `data-is-active` if you need **discrete state changes** (on/off) rather than animated values (0 to 1).

The `data-is-active` attribute is set on the **container element inside the shadow DOM**:

```css
/* ✅ CORRECT - Access via ::part(container) with exact value match */
hover-tilt::part(container)[data-is-active="true"] .my-element {
  background-color: red; /* Discrete state change */
}
```

### ❌ Common Mistakes

```css
/* ❌ WRONG - This won't work with web components */
hover-tilt[data-is-active] .my-element {
  transform: translateZ(50px);
}

/* ❌ WRONG - Missing exact value match */
hover-tilt::part(container)[data-is-active] .my-element {
  transform: translateZ(50px);
}

/* ❌ UNNECESSARY - Use CSS variable instead */
hover-tilt::part(container)[data-is-active="true"] .my-element {
  transform: translateZ(50px); /* Use --hover-tilt-opacity instead! */
}
```

## Shadow DOM Structure

The web component renders with this structure:

```txt
<hover-tilt>                          ← Host element (Light DOM)
  ├── #shadow-root
  │     └── <div part="container">    ← Container with data-is-active
  │           └── <div part="tilt">   ← Tilt layer with transforms
  │                 └── <slot />      ← Your content goes here
  └── slotted content                 ← Light DOM content
```

## Styling Shadow DOM Elements

Use the `::part()` pseudo-element to style internal shadow DOM elements:

```css
/* Target the container */
hover-tilt::part(container) {
  border-radius: 1rem;
  perspective: 800px;
}

/* Target the tilt layer */
hover-tilt::part(tilt) {
  opacity: 0.9;
}

/* Target container when active */
hover-tilt::part(container)[data-is-active="true"] {
  /* styles here */
}
```

## Styling Slotted Content

Your slotted content is in the **Light DOM** and can be styled normally with class selectors:

```html
<hover-tilt class="my-component">
  <div class="my-content">
    <!-- This is Light DOM, style it normally -->
  </div>
</hover-tilt>
```

```css
.my-content {
  /* ✅ This works - slotted content is in Light DOM */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

## CSS Variables in Web Components

CSS variables from the component are exposed to slotted content via `::slotted(*)` internally. You can use them directly in your slotted content:

```css
.my-element {
  /* These CSS variables are available to slotted content */
  opacity: var(--hover-tilt-opacity, 0);
  transform: translateZ(calc(var(--hover-tilt-opacity, 0) * 50px));
}
```

## Common Patterns for Active State

When you need slotted elements to respond to the component's active state, use CSS variables:

```html
<hover-tilt class="stacked-demo">
  <div class="content">
    <h1 class="text">Hello</h1>
    <img class="icon" src="icon.svg" />
  </div>
</hover-tilt>
```

```css
.stacked-3d::part(tilt) {
  /* by default, the will-change list is: transform, box-shadow, opacity */
  will-change: transform, box-shadow, mask;
  /* we remove the 'opacity' property from the will-change list, because when 'opacity'
    is set; the 3d context becomes "flat" and will not allow parallax effects. */
}

.stacked-3d::part(tilt)::before {
  /* make sure the glare is on top of the content */
  z-index: 1;
}

.content {
  /* Enable 3D space */
  transform-style: preserve-3d;
}

/* ✅ BEST APPROACH - Use CSS variables directly */
.text {
  transform: translateZ(calc(var(--hover-tilt-opacity, 0) * 40px));
  transition: transform 0.4s ease-out;
}

.icon {
  transform: translateZ(calc(var(--hover-tilt-opacity, 0) * 80px));
  transition: transform 0.4s ease-out;
}
```

The `--hover-tilt-opacity` variable (and others) are automatically available to slotted content!

## Svelte Component vs Web Component

These styling differences are **specific to the web component**. The Svelte component has different styling considerations:

### Svelte Component

```svelte
<HoverTilt class="my-tilt">
  <div class="content">...</div>
</HoverTilt>

<style>
  /* ✅ Can use :global() to target data-is-active */
  :global([data-is-active="true"]) .content {
    transform: translateZ(50px);
  }
</style>
```

### Web Component

```html
<hover-tilt class="my-tilt">
  <div class="content">...</div>
</hover-tilt>

<style>
  /* ✅ Must use ::part(container) */
  .my-tilt::part(container)[data-is-active="true"] .content {
    transform: translateZ(50px);
  }
</style>
```

## Documentation Reference

Full styling documentation is available at:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simeydotme/hover-tilt](https://github.com/simeydotme/hover-tilt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
