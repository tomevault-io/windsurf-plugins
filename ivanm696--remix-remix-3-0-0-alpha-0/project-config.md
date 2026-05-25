---
trigger: always_on
description: This guide provides a comprehensive overview of the Remix Component API, its runtime behavior, and practical use cases for building interactive UIs.
---

# Remix Component - Agent Guide

This guide provides a comprehensive overview of the Remix Component API, its runtime behavior, and practical use cases for building interactive UIs.

## Getting Started

### Creating a Root

To start using Remix Component, create a root and render your top-level component:

```tsx
import { createRoot } from '@remix-run/component'
import type { Handle } from '@remix-run/component'

function App(handle: Handle) {
  return () => (
    <div>
      <h1>Hello, World!</h1>
    </div>
  )
}

// Create a root attached to a DOM element
let container = document.getElementById('app')!
let root = createRoot(container)

// Render your app
root.render(<App />)
```

The `createRoot` function takes a DOM element (or `document.body`) and returns a root object with a `render` method. You can call `render` multiple times to update the app:

{% raw %}
```tsx
function App(handle: Handle) {
  let count = 0

  return () => (
    <div>
      <h1>Count: {count}</h1>
      <button
        on={{
          click() {
            count++
            handle.update()
          },
        }}
      >
        Increment
      </button>
    </div>
  )
}

let root = createRoot(document.body)
root.render(<App />)

// Later, you can update the app by calling render again
// root.render(<App />)
```
{% endraw %}

### Root Methods

The root object provides several methods:

- **`render(node)`** - Renders a component tree into the root container
- **`flush()`** - Synchronously flushes all pending updates and tasks
- **`remove()`** - Removes the component tree and cleans up

```tsx
let root = createRoot(document.body)

// Render initial app
root.render(<App />)

// Flush any pending updates synchronously
root.flush()

// Later, remove the app
root.remove()
```

## Component Factory and Runtime Behavior

### Component Structure

All components follow a consistent two-phase structure:

1. **Setup Phase** - Runs once when the component is first created
2. **Render Phase** - Runs on initial render and every update afterward

```tsx
function MyComponent(handle: Handle, setup: SetupType) {
  // Setup phase: runs once
  let state = initializeState(setup)

  // Return render function: runs on every update
  return (props: Props) => {
    return <div>{/* render content */}</div>
  }
}
```

### Runtime Behavior

When a component is rendered:

1. **First Render**:

   - The component function is called with `handle` and the `setup` prop
   - The returned render function is stored
   - The render function is called with regular props
   - Any tasks queued via `handle.queueTask()` are executed after rendering

2. **Subsequent Updates**:

   - Only the render function is called
   - Setup phase is skipped, setup closure persists for the lifetime of the component instance
   - Props are passed to the render function
   - The `setup` prop is stripped from props
   - Tasks queued during the update are executed after rendering

3. **Component Removal**:
   - `handle.signal` is aborted
   - All event listeners registered via `handle.on()` are automatically cleaned up
   - Any queued tasks are executed with an aborted signal

### Setup vs Props

The `setup` prop is special - it's only available in the setup phase and is automatically excluded from props. This prevents accidental stale captures:

```tsx
function Counter(handle: Handle, setup: number) {
  // setup prop (e.g., initialCount) only available here
  let count = setup

  return (props: { label: string }) => {
    // props only receives { label } - setup is excluded
    return (
      <div>
        {props.label}: {count}
      </div>
    )
  }
}

// Usage
let element = <Counter setup={10} label="Count" />
```

## Handle API

The `Handle` object provides the component's interface to the framework:

### `handle.update(task?)`

Schedules a component update. Optionally accepts a task to run after the update completes.

```tsx
function Counter(handle: Handle) {
  let count = 0

  return () => (
    <button
      on={{
        click() {
          count++
          handle.update()
        },
      }}
    >
      Count: {count}
    </button>
  )
}
```

With a task:

```tsx
function Player(handle: Handle) {
  let isPlaying = false
  let stopButton: HTMLButtonElement

  return () => (
    <button
      disabled={isPlaying}
      on={{
        click() {
          isPlaying = true
          handle.update(() => {
            // Task runs after update completes
            stopButton.focus()
          })
        },
      }}
    >
      Play
    </button>
  )
}
```

### `handle.queueTask(task)`

Schedules a task to run after the next update. The task receives an `AbortSignal` that's aborted when:

- The component re-renders (new render cycle starts)
- The component is removed from the tree

**Use `queueTask` in event handlers when work needs to happen after DOM changes:**

```tsx
function Form(handle: Handle) {
  let showDetails = false
  let detailsSection: HTMLElement

  return () => (
    <form>
      <input
        type="checkbox"
        checked={showDetails}
        on={{
          change(event) {
            showDetails = event.currentTarget.checked
            handle.update()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivanm696/remix-remix-3.0.0-alpha.0](https://github.com/ivanm696/remix-remix-3.0.0-alpha.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
