---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Overview

UI5 Web Components for React is a React wrapper library for SAP's UI5 Web Components. It provides Fiori-compliant React components by wrapping native UI5 Web Components with React-specific implementations. The project is a monorepo managed with Lerna and Yarn workspaces.

## Critical: Working with Web Components

### Event Handling (MUST KNOW)

Web components emit **CustomEvents** with data in the `detail` property:

```tsx
// ❌ WRONG - won't work with web components
const handleChange = (e) => {
  console.log(e.selectedOption); // undefined!
  console.log(e.value); // undefined!
};

// ✅ CORRECT - access via e.detail
const handleChange = (e) => {
  console.log(e.detail.selectedOption);
  console.log(e.detail.value);
};
```

For enriching events with additional data (internal use):

```tsx
import { enrichEventWithDetails } from '@ui5/webcomponents-react-base/internal/utils';

onLoadMore(enrichEventWithDetails(e, { rowCount, totalRowCount }));
```

### Controlled Components (MUST KNOW)

UI5 Web Components update internal state **before** firing events. For fully controlled components, call `e.preventDefault()` in the handler to prevent internal state updates (e.g., `Input.onInput`, `CheckBox.onChange`, `Select.onChange`, `Dialog.onBeforeClose`).

### Ref Handling (MUST KNOW)

When a component needs **both** a forwarded ref and internal access to the same DOM node, use `useSyncRef()` instead of `useRef()`:

```tsx
import { useSyncRef } from '@ui5/webcomponents-react-base/internal/hooks';

const MyComponent = forwardRef((props, ref) => {
  const [componentRef, internalRef] = useSyncRef(ref);

  useEffect(() => {
    // internalRef gives stable access to the DOM node inside the component
    internalRef.current?.focus();
  }, []);

  // componentRef is passed to the DOM element so both the forwarded ref and internalRef stay in sync
  return <div ref={componentRef} />;
});
```

If the ref is just passed through without internal usage, forward it directly - no `useSyncRef` needed.

### Slot Props (MUST KNOW)

When passing custom React components to slot props, they **must forward the `slot` prop**:

```tsx
// ❌ WRONG - slot won't render correctly
const CustomHeader = ({ children }) => <div>{children}</div>;

// ✅ CORRECT - forwards slot prop to root element
const CustomHeader = ({ children, slot }) => <div slot={slot}>{children}</div>;

// Usage
<Dialog header={<CustomHeader>Title</CustomHeader>} />;
```

**Note:** React Portals are NOT supported in slot props (will log warning).

### CSS and Shadow DOM Styling

Web component internals are in Shadow DOM. Use `::part()` pseudo-element:

```css
/* ❌ WRONG - can't pierce shadow DOM */
.myButton button {
  background: red;
}

/* ✅ CORRECT - use ::part() for shadow DOM */
.myCheckbox::part(root) {
  display: flex;
  width: unset;
}

.actionSheet::part(content) {
  padding: 0.1875rem 0.375rem;
}
```

Use SAP CSS variables directly:

```css
.myComponent {
  background: var(--sapBackgroundColor);
  color: var(--sapTextColor);
  border-radius: var(--sapElement_BorderCornerRadius);
  font-family: var(--sapFontFamily);

  /* Semantic colors */
  --success: var(--sapPositiveColor);
  --error: var(--sapNegativeColor);
  --warning: var(--sapCriticalColor);
  --info: var(--sapInformativeColor);
}
```

Internal component variables use `--_ui5wcr*` prefix:

```tsx
const style = {
  '--_ui5wcr-AnalyticalTableRowHeight': `${rowHeight}px`,
} as CSSProperties;
```

### SSR Compatibility

Components must work in both SPA and SSR environments. Wrappers use `suppressHydrationWarning` because web component internal state doesn't serialize consistently.

### CSS Modules

Component styles use CSS Modules (`.module.css` files). The corresponding `.css.ts` files are **auto-generated** by the build system and **gitignored** (`src/**/*.css.ts` in each package's `.gitignore`). Never create or edit `.css.ts` files manually — only create the `.module.css` source file.

## Core Architecture

### Base Package Imports

The `@ui5/webcomponents-react-base` package uses subpath exports. Always use the specific subpath, not the root import:

```tsx
// ❌ WRONG - don't use root import
import { useSyncRef } from '@ui5/webcomponents-react-base';

// ✅ CORRECT - use subpath exports
import { useI18nBundle, useViewportRange } from '@ui5/webcomponents-react-base/hooks';
import {
  useSyncRef,
  useStylesheet,
  useIsRTL,
  useIsomorphicLayoutEffect,
  useCurrentTheme,
} from '@ui5/webcomponents-react-base/internal/hooks';
import { debounce, throttle, enrichEventWithDetails } from '@ui5/webcomponents-react-base/internal/utils';
import { Device } from '@ui5/webcomponents-react-base/Device';
```

**Available subpath exports:**

- `./hooks` - Public hooks: `useI18nBundle`, `useViewportRange`
- `./internal/hooks` - Internal hooks: `useSyncRef`, `useStylesheet`, `useIsRTL`, `useIsomorphicLayoutEffect`, `useCurrentTheme`
- `./internal/utils` - Utilities: `debounce`, `throttle`, `enrichEventWithDetails`
- `./Device` - Device detection utilities
- `./types` - TypeScript types

### Package Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UI5/webcomponents-react](https://github.com/UI5/webcomponents-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
