---
trigger: always_on
description: **Purpose:** This document provides AI agents with a comprehensive technical overview of the GPUI-TS project. Its goal is to enable you to understand the architecture, APIs, and contribution guidelines, allowing you to safely and effectively read, modify, and extend the codebase.
---

# AGENTS.md: A Developer's Guide for AI Agents

**Project:** GPUI-TS
**Purpose:** This document provides AI agents with a comprehensive technical overview of the GPUI-TS project. Its goal is to enable you to understand the architecture, APIs, and contribution guidelines, allowing you to safely and effectively read, modify, and extend the codebase.

---

## 1. Project Overview & Core Philosophy

**GPUI-TS is a type-safe, schema-driven state management library for web applications.**

Its primary goal is to provide maximum **predictability** and **developer ergonomics** for managing complex state. It achieves this through three core architectural principles:

1.  **Centralized State Ownership:** All application state is defined in a single, unified `AppSchema`. There is one global source of truth, managed by a central `ModelRegistry`. This eliminates state scattered across components.
2.  **Explicit, Atomic Updates:** State is never mutated directly. All modifications are made through explicit functions (e.g., `model.update()`). These updates are designed to be transactional; if they fail, the state should remain consistent.
3.  **Queued Effect System:** State changes do not immediately trigger UI re-renders or other side effects. Instead, they queue "effects" (like notifications or events) which are flushed in a predictable, run-to-completion cycle. This prevents race conditions and cascading updates.

Your primary directive when working with this codebase is to **uphold these principles.**

---

## 2. Key Files & Directory Structure

To work within this project, you must understand the role of its key files.

-   **`src/index.ts` (The Core Engine):**
    -   Contains the `ModelRegistry` class, which is the heart of the framework.
    -   Defines the `createApp` and `createModelAPI` factory functions.
    -   Implements all core `ModelAPI` methods (`update`, `read`, `onChange`, `emit`, etc.).
    -   **When to modify:** Only when changing fundamental mechanics of state management, transactions, or the effect queue.

-   **`src/lit.ts` (Rendering Layer):**
    -   The integration layer for the `lit-html` rendering library.
    -   Contains `createView`, `createComponent`, and various UI helpers like `suspense`.
    -   **When to modify:** When changing how state is connected to the DOM or adding new rendering capabilities.

-   **`src/helpers.ts` (Schema Definition):**
    -   Contains the `createSchema()` fluent builder and other utilities for schema composition.
    -   **When to modify:** When adding new capabilities to the schema-building process itself (e.g., new plugin types, new builder methods).

-   **Advanced Features (`src/advanced.ts`, `src/robot.ts`, `src/crdt.ts`):**
    -   These modules contain high-level abstractions like `createResource`, state machine integrations, and CRDTs. They are built on top of the core primitives.
    -   **When to modify:** When enhancing or fixing these specific advanced features.

-   **`src/ergonomic.ts` (Developer Experience):**
    -   Contains the `unctx`-based context API (`createAppWithContext`, `useModel`). This is the preferred API for end-users.
    -   **When to modify:** To add new context-aware hooks.

-   **`test/` (Verification):**
    -   Contains the `vitest` test suite. Every file in `src/` should have a corresponding test file here.
    -   **When to modify:** ALWAYS. Any change to source code must be accompanied by new or updated tests.

-   **`README.md` & `docs/` (User-Facing Documentation):**
    -   These files explain how to use the library. They should be considered the "specification" for the public API.
    -   **When to modify:** When adding, removing, or changing any public-facing API.

---

## 3. Primary APIs and Usage Patterns

When generating code, you should adhere to these established patterns.

### a. Schema and App Initialization

```typescript
import { createSchema, createApp } from 'gpui-ts';

// Always start by defining a schema.
const AppSchema = createSchema()
  .model('user', { name: '', loggedIn: false })
  .model('todos', { items: [] })
  .build();

// Create the app instance.
const app = createApp(AppSchema);
```

### b. State Access and Mutation

```typescript
// Access a model.
const userModel = app.models.user;

// Read state. This is an immutable clone.
const currentUser = userModel.read();

// Mutate state using the .update() method.
userModel.update((state, ctx) => {
  state.name = 'New Name';
  // Always call ctx.notify() to trigger UI updates.
  ctx.notify();
});

// Use path-based helpers for targeted updates.
userModel.updateAt('name', () => 'Another Name');
```

### c. Subscriptions and Events

```typescript
// Subscribe to state changes.
const unsubscribe = userModel.onChange(newState => {
  console.log('User state changed:', newState);
});

// Emit and listen for typed events.
userModel.onEvent(event => {
  console.log('Event received:', event);
});
userModel.emit({ type: 'USER_LOGIN', timestamp: Date.now() });
```

---

## 4. Development Workflow & Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doeixd/gpui-ts](https://github.com/doeixd/gpui-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
