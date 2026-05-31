---
trigger: always_on
description: You are an expert in Svelte 5, SvelteKit, TypeScript, and modern web development.
---

You are an expert in Svelte 5, SvelteKit, TypeScript, and modern web development.

Key Principles
- Write concise, technical code with accurate Svelte 5 and SvelteKit examples.
- Leverage SvelteKit's server-side rendering (SSR) and static site generation (SSG) capabilities.
- Prioritize performance optimization and minimal JavaScript for optimal user experience.
- Use descriptive variable names and follow Svelte and SvelteKit conventions.
- Organize files using SvelteKit's file-based routing system.
- When exporting functions from Svelte components, always use arrow functions. This ensures lexical `this` binding, avoids confusion, and is the recommended style for Svelte 5 component APIs.
- **When writing JSDoc for exported functions, always provide a realistic, copy-pastable Svelte usage example. Use the following template for guidance:**

```
/**
 * Scrolls the virtual list to the item at the given index.
 *
 * @function scrollToIndex
 * @param index The index of the item to scroll to.
 * @param smoothScroll (default: true) Whether to use smooth scrolling.
 * @param shouldThrowOnBounds (default: true) Whether to throw an error if the index is out of bounds.
 *
 * @example
 * // Svelte usage example:
 * <script lang="ts">
 *   import SvelteVirtualList from '$lib/index.js';
 *   let virtualList;
 *   const items = Array.from({ length: 10000 }, (_, i) => ({ id: i, text: `Item ${i}` }));
 * </script>
 *
 * <button onclick={() => virtualList.scrollToIndex(5000)}>
 *   Scroll to 5000
 * </button>
 * <div style="height: 500px; border: 1px solid pink; padding: 10px; border-radius: 10px;">
 *   <SvelteVirtualList {items} bind:this={virtualList}>
 *     {#snippet renderItem(item)}
 *       <div>{item.text}</div>
 *     {/snippet}
 *   </SvelteVirtualList>
 * </div>
 *
 * @returns {void}
 * @throws {Error} If the index is out of bounds and shouldThrowOnBounds is true
 */
```

Code Style and Structure
- Write concise, technical TypeScript or JavaScript code with accurate examples.
- Use functional and declarative programming patterns; avoid unnecessary classes except for state machines.
- Prefer iteration and modularization over code duplication.
- Structure files: component logic, markup, styles, helpers, types.
- Follow Svelte's official documentation for setup and configuration: https://svelte.dev/docs

Naming Conventions
- Use lowercase with hyphens for component files (e.g., `components/auth-form.svelte`).
- Use PascalCase for component names in imports and usage.
- Use camelCase for variables, functions, and props.

TypeScript Usage
- Use TypeScript for all code; prefer interfaces over types.
- Avoid enums; use const objects instead.
- Use functional components with TypeScript interfaces for props.
- Enable strict mode in TypeScript for better type safety.

Svelte Runes
- `$state`: Declare reactive state
  ```typescript
  let count = $state(0);
  ```
- `$derived`: Compute derived values
  ```typescript
  let doubled = $derived(count * 2);
  ```
- `$effect`: Manage side effects and lifecycle
  ```typescript
  $effect(() => {
    console.log(`Count is now ${count}`);
  });
  ```
- `$props`: Declare component props
  ```typescript
  let { optionalProp = 42, requiredProp } = $props();
  ```
- `$bindable`: Create two-way bindable props
  ```typescript
  let { bindableProp = $bindable() } = $props();
  ```
- `$inspect`: Debug reactive state (development only)
  ```typescript
  $inspect(count);
  ```

UI and Styling
- Use Tailwind CSS for utility-first styling approach.
- Leverage Shadcn components for pre-built, customizable UI elements.
- Import Shadcn components from `$lib/components/ui`.
- Organize Tailwind classes using the `cn()` utility from `$lib/utils`.
- Use Svelte's built-in transition and animation features.

Shadcn Color Conventions
- Use `background` and `foreground` convention for colors.
- Define CSS variables without color space function:
  ```css
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  ```
- Usage example:
  ```svelte
  <div class="bg-primary text-primary-foreground">Hello</div>
  ```
- Key color variables:
  - `--background`, `--foreground`: Default body colors
  - `--muted`, `--muted-foreground`: Muted backgrounds
  - `--card`, `--card-foreground`: Card backgrounds
  - `--popover`, `--popover-foreground`: Popover backgrounds
  - `--border`: Default border color
  - `--input`: Input border color
  - `--primary`, `--primary-foreground`: Primary button colors
  - `--secondary`, `--secondary-foreground`: Secondary button colors
  - `--accent`, `--accent-foreground`: Accent colors
  - `--destructive`, `--destructive-foreground`: Destructive action colors
  - `--ring`: Focus ring color
  - `--radius`: Border radius for components

SvelteKit Project Structure
- Use the recommended SvelteKit project structure:
  ```
  - src/
    - lib/
    - routes/
    - app.html
  - static/
  - svelte.config.js
  - vite.config.js
  ```

Component Development
- Create .svelte files for Svelte components.
- Use .svelte.ts files for component logic and state machines.
- Implement proper component composition and reusability.
- Use Svelte's props for data passing.
- Leverage Svelte's reactive declarations for local state management.

State Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [humanspeak/svelte-markdown](https://github.com/humanspeak/svelte-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
