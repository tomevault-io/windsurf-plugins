---
trigger: always_on
description: You are lead developer for a modular javascript npm package. These instructions give the context to work on the project, follow them **Carefully**.
---

# HellaJS Project Instructions

You are lead developer for a modular javascript npm package. These instructions give the context to work on the project, follow them **Carefully**.

## Persona Guidelines

- You **ALWAYS** build an understanding of the entire project folder structure
- You **ALWAYS** build an understanding of the relationships between entities (packages, plugins, scripts, etc.)
- You **ALWAYS** check and execute the correct CI scripts
- You **ALWAYS** follow your Coding Guidelines
- You **ALWAYS** follow your Testing Guidelines
- You **ALWAYS** follow your Writing Guidelines

## Packages

### Core

High-performance reactive primitives using doubly-linked dependency graphs and topological execution. Implements a directed acyclic graph where signals are sources, computed values are transforms, and effects are sinks. Updates propagate through the graph in topological order with glitch-free guarantees (each node executes max once per update).

**API**:
- `signal()`: Writable reactive state containers
- `computed()`: Derived values that auto-update when dependencies change
- `effect()`: Side effects that run when dependencies change, return cleanup function
- `batch()`: Defer effect execution until batch completes
- `untracked()`: Read signals without creating dependencies

**Example**:
```js
import { signal, computed, effect, batch, untracked } from '@hellajs/core'

// Create signals (writable state)
const count = signal(0)
const multiplier = signal(2)

// Create computed (derived state)
const doubled = computed(() => count() * multiplier())

// Create effect (side effect that auto-runs when dependencies change)
const cleanup = effect(() => {
  console.log(`Count: ${count()}, Doubled: ${doubled()}`)
})

// Batch multiple updates to run effects once
batch(() => {
  count(count() + 1)
  multiplier(3)
})

// Read signal without creating dependency
const currentCount = untracked(() => count())

// Cleanup effect when done
cleanup()
```

**Reference**: `packages/core/CLAUDE.md` for detailed implementation reference

### DOM

Surgical DOM updates without virtual DOM diffing. Only elements with reactive dependencies update, not entire trees. Features automatic cleanup via MutationObserver (auto-disposes effects/events on node removal), global event delegation (single listener per type on document.body), and keyed list reconciliation using LIS algorithm for minimal moves.

**API**:
- `mount()`: Render HellaNode to DOM with reactive bindings
- `forEach()`: Keyed list reconciliation with LIS algorithm
- `element()`: Chainable API for existing DOM elements
- `elements()`: Chainable API for multiple DOM elements

**Example**:
```js
import { signal } from '@hellajs/core'
import { mount, forEach, element, elements } from '@hellajs/dom'

const count = signal(0)
const items = signal([{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }])

// Mount reactive DOM tree
mount(
  <div>
    <h1>Count: {count}</h1>
    <button onClick={() => count(count() + 1)}>Increment</button>

    {/* Keyed list reconciliation with minimal DOM moves */}
    {forEach(items, (item) => (
      <li key={item.id}>{item.name}</li>
    ))}
  </div>
)

// Chainable API for existing DOM elements
element('#myButton')
  .on('click', () => console.log('clicked'))
  .text('Click me')
  .attr({ "class": "btn" })

// Chainable API for multiple elements
elements('.item')
  .attr({ 'data-loaded': 'true' })
```

**Reference**: `packages/dom/CLAUDE.md` for detailed implementation reference

### CSS

Type-safe CSS-in-JS with runtime style generation, automatic memory management, and reactive CSS variables. Generates unique class names and injects styles into the DOM with reference counting for automatic cleanup. Supports reactive CSS custom properties that update when signals change.

**API**:
- `css()`: Create styles and return generated class name
- `cssVars()`: Create CSS custom properties with optional reactivity
- `cssRemove()`: Remove specific styles and decrement reference count
- `cssReset()`: Clear all CSS rules, caches, and reset system
- `cssVarsReset()`: Clear all CSS variables and reactive effects

**Example**:
```js
import { signal } from '@hellajs/core'
import { css, cssVars } from '@hellajs/css'

// Create reactive theme variables
const isDark = signal(false)

// Global styles
css({
  body: { margin: 0, fontFamily: 'system-ui' }
}, { global: true })

// Scoped and prefixed variables
const componentVars = cssVars({
  size: '16px',
  weight: 'bold'
}, { scoped: '.card', prefix: 'ui' })

const theme = cssVars({
  colors: {
    primary: '#3b82f6',
    background: () => isDark() ? '#1a1a1a' : '#ffffff',
    text: () => isDark() ? '#ffffff' : '#000000'
  },
  spacing: '1rem'
})

// Create styles using theme variables
const buttonStyle = css({
  padding: theme.spacing,
  backgroundColor: theme.colors.primary,
  color: theme.colors.text,
  border: 'none',
  borderRadius: '0.5rem',
  cursor: 'pointer',
  ':hover': { opacity: 0.8 }
})

// Use in component
<button class={buttonStyle} onClick={() => isDark(!isDark())}>
  Toggle Theme
</button>

```

**Reference**: `packages/css/CLAUDE.md` for detailed implementation reference

### Resource


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omilli/hellajs](https://github.com/omilli/hellajs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
