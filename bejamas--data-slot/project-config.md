---
trigger: always_on
description: This document captures conventions, patterns, and guidelines for developing new packages in the @data-slot monorepo.
---

# CLAUDE.md - Development Guide for @data-slot

This document captures conventions, patterns, and guidelines for developing new packages in the @data-slot monorepo.

## Project Overview

**Monorepo structure:**
- `packages/` - Component packages and core utilities
- `website/` - Astro-based documentation site

**Package naming:** `@data-slot/{component-name}`

**Current version:** 0.2.10 (synchronized across all packages)

## Package Structure Template

```
packages/{component-name}/
├── src/
│   ├── index.ts          # Main implementation
│   └── index.test.ts     # Bun tests
├── package.json
├── tsdown.config.ts
└── README.md
```

## Core Utilities (@data-slot/core)

All components import from `@data-slot/core`. Available utilities:

### DOM Utilities
```typescript
import { getPart, getParts, getRoots, getDataBool, getDataNumber, getDataString, getDataEnum } from "@data-slot/core";

// Find single part within root
getPart<HTMLElement>(root, "tabs-list")

// Find all parts within root
getParts<HTMLElement>(root, "tabs-trigger")

// Find all root elements in scope
getRoots(scope, "tabs")

// Read typed data attributes (converts kebab-case to camelCase)
getDataString(root, "defaultValue")  // reads data-default-value
getDataNumber(root, "min")           // reads data-min, returns number
getDataBool(root, "disabled")        // reads data-disabled, returns boolean
getDataEnum(root, "orientation", ["horizontal", "vertical"] as const)
```

### ARIA Utilities
```typescript
import { ensureId, setAria, linkLabelledBy } from "@data-slot/core";

// Generate unique ID if element doesn't have one
ensureId(element, "tab")  // Returns existing id or generates "tab-{random}"

// Set ARIA attributes (handles boolean conversion)
setAria(element, "selected", true)   // aria-selected="true"
setAria(element, "expanded", false)  // aria-expanded="false"
setAria(element, "orientation", "vertical")

// Link label to element
linkLabelledBy(element, labelElement)
```

### Event Utilities
```typescript
import { on, emit, composeHandlers } from "@data-slot/core";

// Add event listener, returns cleanup function
const cleanup = on(element, "click", handler)

// Emit custom event
emit(root, "tabs:change", { value: "two" })

// Compose multiple handlers
composeHandlers(handler1, handler2)
```

## Component Implementation Pattern

### Interfaces
```typescript
export interface ComponentOptions {
  /** Initial value */
  defaultValue?: string;
  /** Callback when value changes */
  onValueChange?: (value: string) => void;
  /** Orientation for keyboard navigation */
  orientation?: "horizontal" | "vertical";
}

export interface ComponentController {
  /** Change value programmatically */
  select(value: string): void;
  /** Currently selected value */
  readonly value: string;
  /** Cleanup all event listeners */
  destroy(): void;
}
```

### Factory Function
```typescript
export function createComponent(
  root: Element,
  options: ComponentOptions = {}
): ComponentController {
  // 1. Query parts
  const list = getPart<HTMLElement>(root, "component-list");
  const items = getParts<HTMLElement>(root, "component-item");

  if (!list || items.length === 0) {
    throw new Error("Component requires component-list and at least one component-item");
  }

  // 2. Resolve options: JS > data-* > defaults
  const orientation = options.orientation ?? getDataEnum(root, "orientation", ORIENTATIONS) ?? "horizontal";
  const defaultValue = options.defaultValue ?? getDataString(root, "defaultValue") ?? "";
  const onValueChange = options.onValueChange;

  // 3. State (closure-based, no classes)
  let currentValue = defaultValue;
  const cleanups: Array<() => void> = [];

  // 4. Setup ARIA roles
  list.setAttribute("role", "listbox");
  for (const item of items) {
    item.setAttribute("role", "option");
    ensureId(item, "item");
  }

  // 5. State application function
  const applyState = (value: string, init = false) => {
    if (currentValue === value && !init) return;
    currentValue = value;

    // Update visual state
    for (const item of items) {
      const isSelected = item.dataset.value === value;
      setAria(item, "selected", isSelected);
      item.dataset.state = isSelected ? "active" : "inactive";
    }

    // Emit events (skip on init)
    if (!init) {
      emit(root, "component:change", { value });
      onValueChange?.(value);
    }
  };

  applyState(currentValue, true);

  // 6. Event handlers (add to cleanups array)
  cleanups.push(on(list, "click", (e) => { /* ... */ }));
  cleanups.push(on(list, "keydown", (e) => { /* ... */ }));

  // 7. Inbound event listener
  cleanups.push(on(root, "component:select", (e) => {
    const detail = (e as CustomEvent).detail;
    const value = typeof detail === "string" ? detail : detail?.value;
    if (value) applyState(value);
  }));

  // 8. Return controller
  return {
    select: (value: string) => applyState(value),
    get value() { return currentValue; },
    destroy: () => {
      cleanups.forEach(fn => fn());
      cleanups.length = 0;
      bound.delete(root);
    },
  };
}
```

### Auto-Discovery Function
```typescript
// WeakSet prevents double-binding
const bound = new WeakSet<Element>();

/**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bejamas/data-slot](https://github.com/bejamas/data-slot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
