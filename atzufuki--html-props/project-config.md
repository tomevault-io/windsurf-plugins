---
trigger: always_on
description: provides a declarative props API with reactive signals for building custom
---

# HTML Props Agent Guidelines

HTML Props is a modular web components framework built on Deno/TypeScript. It
provides a declarative props API with reactive signals for building custom
elements. The project is organized as a monorepo with independent packages
published to JSR.

---

## Architecture

### Module Structure

The workspace contains **8 independent packages** in `src/`:

| Package      | Description                                                            |
| ------------ | ---------------------------------------------------------------------- |
| `core`       | Foundation: reactive props mixin (`HTMLPropsMixin`), lifecycle hooks   |
| `signals`    | Reactive primitives: `signal()` and `effect()` for fine-grained reactivity |
| `jsx`        | JSX runtime for template syntax (optional)                             |
| `built-ins`  | Type-safe wrappers for standard HTML elements (Div, Button, etc.)      |
| `layout`     | Flutter-inspired layout components (Row, Column, Stack, etc.)          |
| `builder`    | Visual HTML page building tool for VS Code                             |
| `create`     | CLI scaffolding tool for new projects                                  |
| `landing`    | Demo/documentation website (dev server, HMR client)                    |

Each package has its own `deno.json` and tests.

### Design Patterns

**Mixin-Based Architecture**: Components extend `HTMLPropsMixin(BaseClass)` to
get props + rendering.

- Supports inheritance: `class Box extends HTMLPropsMixin(Widget)` - mixins
  auto-compose.
- Props defined via config object:
  `HTMLPropsMixin(HTMLElement, { count: prop(0) })`.

**Signal-Based Reactivity**: Props backed by signals from `@html-props/signals`.

- Signals are callable: `this.count()` gets value, `this.count = 5` sets value.
- Effects subscribe to signals; components auto-rerender when props change.

**Constructor Props Pattern**: Create elements with props directly:

```typescript
new MyElement({ text: "Hello", count: 5, onclick: () => {} });
```

Constructor accepts props object; applies style, class, content, refs, event
listeners.

**DOM Reconciliation**: Components use a custom reconciliation algorithm for
efficient updates.

- Compares previous and new render output, applies only necessary changes.
- Preserves focus, scroll position, and animation state.
- Uses `dataset.key` for stable element identity in lists.
- `forceUpdate()` bypasses reconciliation for full re-render when needed.

---

## File Naming Conventions

**All TypeScript files must use lowercase with underscores:**

- ✅ `props_controller.ts`
- ✅ `html_props_mixin.ts`
- ❌ `PropsController.ts`
- ❌ `HTMLPropsMixin.ts`

---

## Developer Workflows

### Testing

```bash
deno task test
```

Tests run in Playwright (real Chromium browser). Each package has isolated
tests.

### Development/Landing Page

```bash
deno task dev
```

Runs HMR dev server, watches `src/landing/` for changes, bundles output.

### Package Publishing

- Root `publish.json` lists packages for JSR: core, create, jsx, signals.
- `built-ins` and `landing` are not published.
- Each package is versioned independently.

---

## Code Patterns

### Defining Custom Elements

```typescript
import { HTMLPropsMixin, prop } from "@html-props/core";

class MyElement extends HTMLPropsMixin(HTMLElement, {
  text: prop("default"),
}) {
  render() {
    return document.createTextNode(this.text);
  }
}

MyElement.define("my-element");
```

- `render()` returns `Node | Node[] | null`. Called whenever props change.
- Optional: `mountedCallback()` and `unmountedCallback()` lifecycle hooks.
- Props are inferred from the config object passed to the mixin.

### Props Configuration

```typescript
class MyElement extends HTMLPropsMixin(HTMLElement, {
  count: prop(0, { attribute: true }),
  label: prop("", { attribute: "data-label", event: "labelChange" }),
  active: prop(false),
  items: prop<string[]>([], { type: Array }),
}) {}
```

| Option      | Description                                              |
| ----------- | -------------------------------------------------------- |
| `type`      | String, Number, Boolean, Array, or Object constructors   |
| `default`   | Initial value (required for type inference)              |
| `attribute` | Boolean (reflect as kebab-case) or String (custom name)  |
| `event`     | Emit custom event on prop change                         |

### Using Signals

```typescript
import { signal, effect } from "@html-props/signals";

const count = signal(0);
effect(() => {
  console.log("Count changed to:", count()); // Tracked dependency
});

count(5); // Triggers effect
count.update((v) => v + 1); // Increment with prev value
```

Signals are fine-grained; effects only re-run if their dependencies changed.

### JSX Usage (Optional)

Configure `deno.json` compiler options:

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "@html-props/jsx"
  }
}
```

Then write JSX in components:

```typescript
render() {
  return <div>{this.count()}</div>;
}
```

---

## List Rendering with Keys

**Always use `dataset.key` for list items** to help the reconciler match DOM
nodes correctly:

```typescript
render() {
  return new Ul({
    content: this.items.map(item =>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atzufuki/html-props](https://github.com/atzufuki/html-props) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
