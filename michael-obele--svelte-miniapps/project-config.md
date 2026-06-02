---
trigger: always_on
description: A modern Svelte 5 PWA featuring multiple single-purpose mini-apps built on SvelteKit with TypeScript, Tailwind CSS, and Prisma.
---

# Svelte MiniApps Instructions

A modern Svelte 5 PWA featuring multiple single-purpose mini-apps built on SvelteKit with TypeScript, Tailwind CSS, and Prisma.

## Project Overview

This is a collection of productivity mini-apps including:

- Budget Tracker, Currency Converter, Todo List
- Markdown Editor, QR Code Generator, Password Generator
- Text Summarizer, Dictionary App, Unit Converter
- GitHub Contribution Tracker, Advanced Emoji Tools

## Quick Start Commands

### Development

- **Install**: `bun install`
- **Dev**: `bun run dev` (port 5178)
- **Dev with Service Worker**: `bun run dev:with-sw`
- **Typecheck**: `bun check` (preferred over `npx tsc`)

### Testing & Quality

- **Tests**: `npm run test` (unit + e2e)
- **Unit Tests**: `npm run test:unit`
- **E2E Tests**: `npm run test:e2e`
- **Watch Tests**: `npm run test:watch`
- **Lint**: `npm run lint`
- **Format**: `npm run format`
- **Typecheck**: `bun check` (preferred over `npx tsc` and lint for code validation)

### Build & Deploy

- **Build**: `bun run build`
- **Preview**: `vite preview`

### Database

- **Studio**: `npm run db:studio`
- **Push**: `npm run db:push`
- **Migrate**: `npm run db:migrate`

## Project Structure

```
src/
├── routes/                     # Pages & API endpoints
│   ├── apps/(app)/            # Individual mini-apps
│   ├── (auth)/                # Authentication pages
│   ├── api/                   # API endpoints
│   └── +layout.svelte         # Root layout
├── lib/
│   ├── components/
│   │   ├── ui/                # shadcn-svelte components
│   │   └── blocks/            # Layout components (Navbar, Footer)
│   ├── server/                # Server-side utilities
│   ├── utility/               # Client utilities
│   └── stores/                # Svelte stores
├── types/                     # TypeScript definitions
└── app.css                    # Global styles
prisma/                        # Database schema & migrations
static/                        # Static assets & PWA files
```

## Svelte 5 Rules (Required)

### Modern Svelte 5 Syntax

**Reactive Variables:**

```svelte
// ✅ Use $state() for reactive variables
let count = $state(0);
let user = $state({ name: '', email: '' });

// ❌ Don't use legacy reactive statements
// $: count = 0;
```

**Component Props:**

```svelte
// ✅ Use $props() to destructure component props
let { title, description, children } = $props();

// ✅ With types
interface Props {
  title: string;
  description?: string;
  children?: Snippet;
}
let { title, description, children }: Props = $props();
```

**Computed Values:**

```svelte
// ✅ Use $derived() for computed values
let doubled = $derived(count * 2);
let fullName = $derived(`${firstName} ${lastName}`);

// ✅ Use $derived.by() for complex computations
let expensiveCalculation = $derived.by(() => {
  // Complex logic here
  return items.filter(i => i.active).reduce((sum, i) => sum + i.value, 0);
});

// ❌ Don't use legacy reactive statements
// $: doubled = count * 2;
```

**Side Effects (Use Sparingly!):**

```svelte
// ⚠️ IMPORTANT: Prefer $derived() over $effect() whenever possible
// Only use $effect() for TRUE side effects (not for computing values)

// ✅ Use $effect() ONLY for side effects like logging, analytics, DOM manipulation
$effect(() => {
  console.log('Count changed:', count);
  // Or: trackAnalytics(count);
  // Or: updateExternalDOM(element, count);
});

// ✅ With cleanup for subscriptions, timers, listeners
$effect(() => {
  const interval = setInterval(() => {
    count++;
  }, 1000);

  return () => clearInterval(interval);
});

// ❌ NEVER assign to reactive state inside $effect() without guards
// This creates infinite loops!
$effect(() => {
  derived = count * 2; // ❌ WRONG - use $derived() instead
});
```

**Reactive State Priority:**

1. **First choice**: `$state()` for mutable values
2. **Second choice**: `$derived()` or `$derived.by()` for computed values
3. **Last resort**: `$effect()` only for true side effects (logging, DOM, external APIs)

**Event Handling:**

```svelte
<!-- ✅ Use onclick={handler} for events -->
<button onclick={() => count++}>Click me</button>
<button onclick={handleClick}>Click me</button>

<!-- ❌ Don't use deprecated on:event syntax -->
<!-- <button on:click={handleClick}>Click me</button> -->
```

**Component Initialization:**

```svelte
// ✅ Use mount() for component initialization
import { mount } from 'svelte';

mount(() => {
  // Component mounted
});
```

### Deprecated Patterns to Avoid

- ❌ Legacy `$:` reactive statements → Use `$state()`, `$derived()`, `$effect()`
- ❌ `on:click="string"` or `on:event` directive syntax → Use `onclick={fn}`
- ❌ `createEventDispatcher()` → Prefer callback props
- ❌ `SvelteComponentTyped` → Use `Component` types instead

## Svelte Development Tools

### Svelte MCP for Documentation

- When you need to look up Svelte or SvelteKit documentation, use the Svelte MCP tools.
- Use `list_sections` to see all available documentation sections.
- Use `get_documentation` with the section name to get the full documentation.

### Svelte Autofixer

- After making any changes to Svelte files, you must run the `svelte_autofixer` tool.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Michael-Obele/Svelte-MiniApps](https://github.com/Michael-Obele/Svelte-MiniApps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
