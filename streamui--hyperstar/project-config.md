---
trigger: always_on
description: > **Very Beta** - API changes frequently. Great for prototypes and fun real-time multiplayer apps.
---

# Hyperstar

> **Very Beta** - API changes frequently. Great for prototypes and fun real-time multiplayer apps.

A server-driven UI framework for Bun. Server owns state, clients sync automatically via SSE.

**Real-time = all clients see the same store.** User A makes a change, User B sees it instantly.

## Core Concept

```
Store (server state, shared)  →  view(ctx) → HTML
         ↓                              ↓
   Broadcast to ALL clients      Signals (client state, private)
         ↓                              ↓
   Action → Update store         Instant UI updates (no roundtrip)
```

**One store. One view function. All clients stay in sync automatically.**

## Architecture

- **JSX rendering**: Uses @kitajs/html with custom `$` prop for reactive attributes
- **Real-time sync**: SSE streaming and Idiomorph DOM morphing
- **State management**: Immutable updates via `ctx.update()`
- **Validation**: Effect Schema for typed action args
- **Runtime**: Built for Bun

## Project Structure

```
packages/hyperstar/src/
├── index.ts          # Main entry, exports createHyperstar, hs, Schema
├── server.ts         # Bun server, SSE handling, action dispatch, signal handles
├── hs.ts             # HSBuilder and hs namespace for reactive attributes
├── jsx-runtime.ts    # Custom JSX runtime for $ prop
├── jsx.d.ts          # JSX type extensions
├── action/
│   ├── index.ts      # Action creation and execution
│   └── schema.ts     # Effect Schema integration
├── core/
│   └── lifecycle.ts  # Lifecycle hooks (onStart, onConnect, etc.)
├── schedule/
│   └── index.ts      # Scheduling helpers (repeat, cron)
└── triggers/
    └── index.ts      # Store change watchers

packages/hyperstar-client/src/
├── index.ts          # Main entry, Hyperstar global
├── actions.ts        # dispatch() - send actions to server
├── signals.ts        # Preact Signals for client state
├── sse.ts            # SSE connection, auto-reconnect
├── morph.ts          # Idiomorph for DOM diffing
├── process.ts        # Process hs-* attributes
└── expression.ts     # Evaluate expressions with signal context

examples/
├── simple-counter.tsx    # Minimal counter
├── counter.tsx           # Counter with form input
├── todos.tsx             # Full todo app with filters
├── chat-room.tsx         # Real-time multi-user chat
├── fps-jsx.tsx           # Timer/FPS stress test
├── state-types.tsx       # Three-tier state demo
├── persistent-notes.tsx  # JSON file persistence
└── sqlite-notes.tsx      # SQLite persistence
```

## API Reference

### Creating an App

```tsx
import { createHyperstar, hs, Schema } from "hyperstar"

interface Todo {
  id: string
  text: string
  done: boolean
}

interface Store {
  todos: Todo[]
}

interface Signals {
  filter: "all" | "active" | "done"
  text: string
  editingId: string | null
}

// Create typed factory with Store, UserStore, and Signals type parameters
const app = createHyperstar<Store, {}, Signals>()

// Get typed signal handles
const { filter, text, editingId } = app.signals

// Actions (server-side state changes)
const addTodo = app.action("addTodo", { text: Schema.String }, (ctx, { text: t }) => {
  ctx.update((s) => ({
    ...s,
    todos: [...s.todos, { id: crypto.randomUUID(), text: t, done: false }],
  }))
  ctx.patchSignals({ text: "" }) // Clear input for triggering user only
})

const toggleTodo = app.action("toggleTodo", { id: Schema.String }, (ctx, { id }) => {
  ctx.update((s) => ({
    ...s,
    todos: s.todos.map((t) => (t.id === id ? { ...t, done: !t.done } : t)),
  }))
})

// App config
app.app({
  store: { todos: [] },
  signals: { filter: "all", text: "", editingId: null },

  view: (ctx) => (
    <div id="app">
      {/* Form with signal binding */}
      <form $={hs.form(addTodo)}>
        <input name="text" $={hs.bind(text)} />
        <button type="submit">Add</button>
      </form>

      {/* Hybrid filtering (server data + client filter) */}
      {ctx.store.todos.map((todo) => (
        <div
          id={`todo-${todo.id}`}
          hs-show={filter.is("all")
            .or(filter.is("active").and(!todo.done))
            .or(filter.is("done").and(todo.done))}
        >
          <input
            type="checkbox"
            checked={todo.done}
            $={hs.action(toggleTodo, { id: todo.id })}
          />
          {todo.text}
        </div>
      ))}
    </div>
  ),
}).serve({ port: 3000 })
```

### Signal Types

Signals are defined as a type parameter and values provided in `app()`:

```tsx
interface Signals {
  // Simple types
  isAdding: boolean
  text: string
  localCounter: number

  // Union types for enums
  filter: "all" | "active" | "done"

  // Nullable types
  editingId: string | null
}

const app = createHyperstar<Store, {}, Signals>()

// Get typed signal handles
const { isAdding, text, filter, editingId } = app.signals

// Provide default values
app.app({
  store: { ... },
  signals: {
    isAdding: false,
    text: "",
    localCounter: 0,
    filter: "all",
    editingId: null,
  },
  view: ...
})
```

### Signal Handle Methods

Signal handles produce client-side JavaScript expressions:

```tsx
// String/enum signal
filter.is("active")        // "$filter.value === 'active'"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StreamUI/hyperstar](https://github.com/StreamUI/hyperstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
