---
trigger: always_on
description: handleSubmit = () => {
---

# GXT (Glimmer-Next) Development Guide

This guide provides comprehensive instructions for AI agents and developers on how to write frontend applications using GXT (glimmer-next).

## Table of Contents

1. [Overview](#overview)
2. [Project Setup](#project-setup)
3. [Component Patterns](#component-patterns)
4. [Reactivity System](#reactivity-system)
5. [Template Syntax](#template-syntax)
6. [Control Flow](#control-flow)
7. [Modifiers and Helpers](#modifiers-and-helpers)
8. [Context API](#context-api)
9. [Suspense and Lazy Loading](#suspense-and-lazy-loading)
10. [Testing](#testing)
11. [SSR and Rehydration](#ssr-and-rehydration)
12. [Architecture Patterns](#architecture-patterns)
13. [Best Practices](#best-practices)
14. [API Reference](#api-reference)
15. [Example: Complete Component](#example-complete-component)

---

## Overview

GXT is a modern, compilable runtime environment designed as a Glimmer-VM alternative. It features:

- Hot Module Replacement (HMR)
- Native shadow-dom support
- Server-Side Rendering (SSR) with rehydration
- Built-in reactivity system
- 40% performance improvement over GlimmerVM
- 2x less memory usage
- Runtime code tree-shaking
- TypeScript support with Glint integration

---

## Project Setup

### Installation

```bash
pnpm create vite my-app --template vanilla-ts
pnpm install @lifeart/gxt
```

### Vite Configuration

```typescript
// vite.config.mts
import { defineConfig } from "vite";
import { compiler } from "@lifeart/gxt/compiler";

export default defineConfig(({ mode }) => ({
  plugins: [compiler(mode)],
}));
```

### TypeScript Configuration (Glint)

```json
{
  "compilerOptions": {
    // your options
  },
  "glint": {
    "environment": "glint-environment-gxt"
  }
}
```

### Entry Point

```typescript
import { renderComponent } from "@lifeart/gxt";
import App from "./App.gts";

const instance = renderComponent(App, {
  args: { name: "My App" },
  element: document.getElementById("app"),
});

// To destroy
import { destroyElement } from "@lifeart/gxt";
destroyElement(instance);
```

---

## Component Patterns

### Class-Based Components

```typescript
import { Component } from "@lifeart/gxt";

type MyComponentArgs = {
  Args: {
    name: string;
    onSubmit?: (value: string) => void;
  };
  Blocks: {
    default: [];
  };
  Element: HTMLDivElement;
};

export class MyComponent extends Component<MyComponentArgs> {
  // Computed property
  get displayName() {
    return `Hello, ${this.args.name}`;
  }

  // Event handler (arrow function to preserve `this`)
  handleSubmit = () => {
    this.args.onSubmit?.("submitted");
  };

  <template>
    <div>
      <h1>{{this.displayName}}</h1>
      <button {{on "click" this.handleSubmit}}>Submit</button>
      {{yield}}
    </div>
  </template>
}
```

**Key Points:**
- Extend `Component<T>` with type signature
- Define `Args`, `Blocks` (slots), and `Element` types
- Access arguments via `this.args`
- Use arrow functions for event handlers
- Template block defines the component's UI

### Function Components

```typescript
import type { ComponentLike } from "@lifeart/gxt";

type ButtonSignature = {
  Args: {
    onClick?: () => void;
  };
  Blocks: {
    default: [];
  };
  Element: HTMLButtonElement;
};

export const Button: ComponentLike<ButtonSignature> = <template>
  <button {{on "click" @onClick}} type="button" ...attributes>
    {{yield}}
  </button>
</template>;
```

**Key Points:**
- Simpler syntax for stateless components
- Use `@` prefix for arguments (e.g., `@onClick`)
- `{{yield}}` renders block content
- `...attributes` spreads passed attributes

### Template-Only Components

```typescript
// Simple.gts
<template>
  <div class="simple">{{@message}}</div>
</template>
```

---

## Reactivity System

### Cell (Mutable State)

```typescript
import { cell, type Cell } from "@lifeart/gxt";

// Create a cell
const count = cell(0);

// Read value
console.log(count.value); // 0

// Update value
count.update(1);
// or
count.value = 1;

// In templates, cells auto-unwrap
<template>
  <div>Count: {{count}}</div>
</template>
```

### CellFor (Object Property Tracking)

```typescript
import { cellFor } from "@lifeart/gxt";

const user = { name: "Alice", age: 30 };

// Create reactive cell for property
const nameCell = cellFor(user, "name");

// Bidirectional binding
nameCell.update("Bob");
console.log(user.name); // "Bob"

user.name = "Carol";
console.log(nameCell.value); // "Carol"
```

### @tracked Decorator

```typescript
import { Component } from "@lifeart/gxt";
import "decorator-transforms/globals"; // Required at top level!

export class Counter extends Component {
  @tracked count = 0;

  increment = () => {
    this.count++; // Automatically triggers re-render
  };

  <template>
    <div>{{this.count}}</div>
    <button {{on "click" this.increment}}>+</button>
  </template>
}
```

**Important:** Add `import 'decorator-transforms/globals';` to your entry file.

---

## Template Syntax

### Interpolation

```gts
<template>
  {{! Static values }}
  <div>Hello, {{@name}}</div>

  {{! Computed values }}
  <div>{{this.computedValue}}</div>

  {{! Helper expressions }}
  <div>{{if @isActive "Active" "Inactive"}}</div>
</template>
```

### HTML Attributes

```gts
<template>
  {{! Static attributes }}
  <div class="container"></div>

  {{! Dynamic attributes }}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lifeart/glimmer-next](https://github.com/lifeart/glimmer-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
