---
trigger: always_on
description: > **Note:** This file is the authoritative source for coding agent instructions.
---

# AGENTS.md

> **Note:** This file is the authoritative source for coding agent instructions.
> If in doubt, prefer AGENTS.md over README.md.

## 🚦 Quick Reference

- **Install dependencies:** `bun install`
- **Start dev server:** `bun run dev`
- **Build for production:** `bun run build`
- **Preview build:** `bun run preview`
- **Search code:** `rg "pattern"`
- **Add package:** `bun add <package-name>`

**Note**: Testing with Vitest can be added later.

---

This file provides comprehensive guidance for coding agents when working with
Lit web components and Tailwind CSS v4 in this starter template.

## Core Development Philosophy

### KISS (Keep It Simple, Stupid)

Simplicity should be a key goal in design. Choose straightforward solutions over
complex ones whenever possible. Simple solutions are easier to understand,
maintain, and debug.

### YAGNI (You Aren't Gonna Need It)

Avoid building functionality on speculation. Implement features only when they
are needed, not when you anticipate they might be useful in the future.

### HTML-first

Prioritize HTML structure and semantics in your components. Use native HTML
elements with proper ARIA attributes when needed. Shadow DOM provides
encapsulation, but your markup should still be semantic and accessible.

### Design Principles

- **Component Encapsulation**: Use Shadow DOM for style and DOM isolation
- **Reactive Properties**: Leverage Lit's reactive system for automatic updates
- **Type Safety**: Use TypeScript with decorators for compile-time correctness
- **Single Responsibility**: Each component should have one clear purpose
- **Composability**: Build complex UIs by composing simple, reusable components

## 🧱 Project Structure & Library Architecture

This is a **starter template** for building web component libraries with Lit and
Tailwind CSS v4.

### Directory Structure

```plaintext
.
├── lib/                         # Library source code
│   ├── assets/                  # Static assets (images, icons)
│   ├── components/              # Lit components (*.ts)
│   ├── shared/                  # Shared utilities and mixins
│   │   ├── tailwindMixin.ts     # TW mixin for Shadow DOM
│   │   ├── tailwindMixin.d.ts   # Type definitions
│   │   └── utils.ts             # Utility functions (cn, etc.)
│   ├── styles/                  # Global styles
│   │   └── tailwind.global.css  # Tailwind configuration
│   └── main.ts                  # Library entry point
├── src/                         # Development playground
│   ├── index.html               # Dev server entry
│   ├── index.css                # Dev styles
│   └── vite-env.d.ts            # Vite type definitions
├── public/                      # Public assets for dev server
├── dist/                        # Build output (gitignored)
├── vite.config.js               # Vite configuration
├── tsconfig.json                # TypeScript configuration
└── package.json                 # Project dependencies
```

### Build Output

The build produces:

- `dist/my-element.js` - ES module
- `dist/my-element.umd.cjs` - UMD module
- `dist/my-element.d.ts` - TypeScript definitions

### Key Files

- **lib/main.ts**: Export all components here for library consumers
- **lib/shared/tailwindMixin.ts**: Mixin to apply Tailwind to Shadow DOM
- **lib/shared/utils.ts**: Helper functions like `cn()` for class merging
- **lib/styles/tailwind.global.css**: Tailwind v4 configuration with `@theme`

## Lit & Web Components

### Core Concepts

Lit is a lightweight library for building fast web components. It provides:

- **Reactive properties** that trigger re-renders
- **Efficient rendering** using lit-html templates
- **Shadow DOM** integration for encapsulation
- **Lifecycle hooks** for component behavior

### Component Structure

**TypeScript (Preferred):**

```typescript
import { LitElement, html, css } from "lit";
import { customElement, property } from "lit/decorators.js";
import { TW } from "../shared/tailwindMixin";

const TwLitElement = TW(LitElement);

@customElement("my-component")
export class MyComponent extends TwLitElement {
  @property({ type: String }) name = "World";
  @property({ type: Number }) count = 0;

  static styles = css`
    :host {
      display: block;
    }
  `;

  render() {
    return html`
      <div class="p-4">
        <h1>Hello, ${this.name}!</h1>
        <button
          @click=${this._increment}
          class="bg-blue-500 text-white px-4 py-2"
        >
          Count: ${this.count}
        </button>
      </div>
    `;
  }

  private _increment() {
    this.count++;
  }
}

declare global {
  interface HTMLElementTagNameMap {
    "my-component": MyComponent;
  }
}
```

### Reactive Properties

```typescript
@property({ type: String }) name = 'default';
@property({ type: Number }) count = 0;
@property({ type: Boolean, reflect: true }) active = false;
@property({ attribute: false }) data = {};  // No attribute binding
@state() private _internal = '';  // Internal state
```

**Key options**: `type`, `reflect`, `attribute`, `converter`. See
[Lit docs](https://lit.dev/docs/components/properties/) for details.

### Lifecycle & Querying

```typescript
// Lifecycle hooks (always call super first)
connectedCallback() { super.connectedCallback(); }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lloydrichards/base_lit-with-tailwind](https://github.com/lloydrichards/base_lit-with-tailwind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
