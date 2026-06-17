---
trigger: always_on
description: Always use CSS utility functions with prefixed classes, never inline styles
---


# CSS Utilities and Class Prefix Rule

**CRITICAL: Never use inline styles. Always use CSS utility functions with the `plugin-name-` prefix.**

## 🚨 **Zero Tolerance for Inline Styles**

Inline styles (`element.style.*`) are **FORBIDDEN** in all TypeScript/JavaScript code.

### ❌ **NEVER Use Inline Styles**

```typescript
// ❌ FORBIDDEN: Any inline style manipulation
element.style.display = "none";
element.style.cssText = "color: red; font-size: 14px;";
element.style.opacity = "0.5";
element.style.visibility = "hidden";
element.style.backgroundColor = "#fff";
element.style.setProperty("color", "red");

// ❌ FORBIDDEN: Even for dynamic values
element.style.width = `${width}px`;
element.style.top = `${position}px`;
```

## ✅ **ALWAYS Use CSS Utility Functions**

### **CSS Prefix Configuration**

Define the prefix once and use these utility functions throughout your codebase:

```typescript
const CSS_PREFIX = "plugin-name-";

/**
 * Prefixes class names with the standard plugin prefix.
 * Handles multiple class names and automatically adds the prefix.
 */
export function cls(...classNames: string[]): string {
  return classNames
    .flatMap((name) => name.split(/\s+/))
    .filter((name) => name.length > 0)
    .map((name) => `${CSS_PREFIX}${name}`)
    .join(" ");
}

/**
 * Adds prefixed class names to an element.
 */
export function addCls(element: HTMLElement, ...classNames: string[]): void {
  const classes = cls(...classNames);
  if (classes) {
    element.classList.add(...classes.split(/\s+/));
  }
}

/**
 * Removes prefixed class names from an element.
 */
export function removeCls(element: HTMLElement, ...classNames: string[]): void {
  const classes = cls(...classNames);
  if (classes) {
    element.classList.remove(...classes.split(/\s+/));
  }
}

/**
 * Toggles prefixed class names on an element.
 */
export function toggleCls(element: HTMLElement, className: string, force?: boolean): boolean {
  return element.classList.toggle(cls(className), force);
}

/**
 * Checks if element has a prefixed class.
 */
export function hasCls(element: HTMLElement, className: string): boolean {
  return element.classList.contains(cls(className));
}
```

## **Utility Function Reference**

### `cls(...classNames: string[]): string`

Prefixes class names and handles multiple classes.

```typescript
// ✅ Single class
cls("calendar-view") // => "plugin-name-calendar-view"

// ✅ Multiple classes
cls("button", "active") // => "plugin-name-button plugin-name-active"

// ✅ Space-separated classes
cls("modal calendar") // => "plugin-name-modal plugin-name-calendar"

// ✅ Mixed usage
cls("button large", "primary") // => "plugin-name-button plugin-name-large plugin-name-primary"
```

### `addCls(element: HTMLElement, ...classNames: string[]): void`

Adds prefixed classes to an element.

```typescript
// ✅ Add single class
addCls(element, "active");

// ✅ Add multiple classes
addCls(element, "modal", "visible");

// ✅ Add space-separated classes
addCls(element, "button primary large");
```

### `removeCls(element: HTMLElement, ...classNames: string[]): void`

Removes prefixed classes from an element.

```typescript
// ✅ Remove single class
removeCls(element, "active");

// ✅ Remove multiple classes
removeCls(element, "loading", "disabled");
```

### `toggleCls(element: HTMLElement, className: string, force?: boolean): boolean`

Toggles a prefixed class on an element.

```typescript
// ✅ Toggle class
toggleCls(element, "active");

// ✅ Force add
toggleCls(element, "visible", true);

// ✅ Force remove
toggleCls(element, "hidden", false);

// ✅ Use return value
const isActive = toggleCls(button, "active");
```

### `hasCls(element: HTMLElement, className: string): boolean`

Checks if element has a prefixed class.

```typescript
// ✅ Check for class
if (hasCls(element, "active")) {
  // Element has "plugin-name-active" class
}

// ✅ Conditional logic
const isVisible = hasCls(modal, "visible");
```

## **Common Pattern Replacements**

### Show/Hide Elements

```typescript
// ❌ WRONG: Inline style
element.style.display = "none";
element.style.display = "block";

// ✅ CORRECT: CSS class
// Define in styles.css:
// .plugin-name-hidden { display: none; }
// .plugin-name-visible { display: block; }
addCls(element, "hidden");
removeCls(element, "hidden");
toggleCls(element, "visible");
```

### Active/Selected States

```typescript
// ❌ WRONG: Inline style
element.style.backgroundColor = "#007acc";
element.style.color = "white";

// ✅ CORRECT: CSS class
// Define in styles.css:
// .plugin-name-active { background-color: #007acc; color: white; }
addCls(element, "active");
removeCls(element, "active");
toggleCls(element, "active");
```

### Visibility Control

```typescript
// ❌ WRONG: Inline style
element.style.opacity = "0";
element.style.visibility = "hidden";

// ✅ CORRECT: CSS class
// Define in styles.css:
// .plugin-name-fade-out { opacity: 0; }
// .plugin-name-invisible { visibility: hidden; }
addCls(element, "fade-out");
addCls(element, "invisible");
```

### Dynamic Positioning

```typescript
// ❌ WRONG: Inline style for static positions
element.style.position = "absolute";
element.style.top = "10px";
element.style.left = "20px";

// ✅ CORRECT: CSS class for static positions
// Define in styles.css:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Real1tyy/BasesImprovements](https://github.com/Real1tyy/BasesImprovements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
