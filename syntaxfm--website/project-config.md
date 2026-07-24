---
trigger: always_on
description: This file contains custom instructions for GitHub Copilot. These instructions are based on the `.cursorrules` file and provide guidelines for Svelte 5, SvelteKit, and TypeScript development.
---

# GitHub Copilot Instructions

This file contains custom instructions for GitHub Copilot. These instructions are based on the `.cursorrules` file and provide guidelines for Svelte 5, SvelteKit, and TypeScript development.

You are an expert in Svelte 5, SvelteKit, TypeScript, and modern web development.

Key Principles

- Write concise, technical code with accurate Svelte 5 and SvelteKit examples.
- Leverage SvelteKit's server-side rendering (SSR) and static site generation (SSG) capabilities.
- Prioritize performance optimization and minimal JavaScript for optimal user experience.
- Use descriptive variable names and follow Svelte and SvelteKit conventions.
- Organize files using SvelteKit's file-based routing system.

Code Style and Structure

- Write concise, technical TypeScript or JavaScript code with accurate examples.
- Use functional and declarative programming patterns; avoid unnecessary classes except for state machines.
- Prefer iteration and modularization over code duplication.
- Structure files: types, component logic, markup, styles.
- Follow Svelte's official documentation for setup and configuration: https://svelte.dev/docs/svelte/overview

Naming Conventions

- Use PascalCase for component files (e.g., `components/AuthForm.svelte`).
- Use PascalCase for component names in imports and usage.
- Use snake_case for variables, functions, and props.
- Use UPPER_CASE for true constants but not general const vars

TypeScript Usage

- Use TypeScript for all code; prefer interfaces over types.
- Avoid enums; use string literals instead and const objects if needed.
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

- Use css vars defined in `src/variables.css` for css vars.
- Prefer css vars for any property where they exist. Colors specifically cannot be defined without css vars.
- Use Svelte's built-in transition and animation features.
- Use `bg` and `fg` convention for background and color properties respecitvely.

SvelteKit Project Structure

- Use the recommended SvelteKit project structure:
  ```
  - src/
    - lib/
    - routes/
  	- assets/
  	- server/
  	- actions/
    - app.html
  - static/
  - svelte.config.js
  - vite.config.js
  ```

Component Development

- Create .svelte files for Svelte components.
- Use .svelte.ts files for state.
- Implement proper component composition and reusability.
- Use Svelte's props for data passing.
- Leverage Svelte's reactive declarations for local state management.

State Management

- Use classes for complex state management (state machines):

  ```typescript
  // counter.svelte.ts
  class Counter {
  	count = $state(0);
  	incrementor = $state(1);

  	increment() {
  		this.count += this.incrementor;
  	}

  	resetCount() {
  		this.count = 0;
  	}

  	resetIncrementor() {
  		this.incrementor = 1;
  	}
  }

  export const counter = new Counter();
  ```

- Use in components:

  ```svelte
  <script lang="ts">
  	import { counter } from './counter.svelte.ts';
  </script>

  <button on:click={() => counter.increment()}>
  	Count: {counter.count}
  </button>
  ```

Routing and Pages

- Utilize SvelteKit's file-based routing system in the src/routes/ directory.
- Implement dynamic routes using [slug] syntax.
- Use load functions for server-side data fetching and pre-rendering.
- Implement proper error handling with +error.svelte pages.

Server-Side Rendering (SSR) and Static Site Generation (SSG)

- Leverage SvelteKit's SSR capabilities for dynamic content.
- Implement SSG for static pages using prerender option.
- Use the adapter-auto for automatic deployment configuration.

Performance Optimization

- Leverage Svelte's compile-time optimizations.
- Use `{#key}` blocks to force re-rendering of components when needed.
- Implement code splitting using dynamic imports for large applications.
- Profile and monitor performance using browser developer tools.
- Minimize use of client-side JavaScript; leverage SvelteKit's SSR and SSG.
- Use modern html elements and features wherever possible.
- Implement proper lazy loading for images and other assets.

Data Fetching and API Routes

- Use load functions for server-side data fetching.
- Implement proper error handling for data fetching operations.
- Create API routes in the src/routes/api/ directory.
- Implement proper request handling and response formatting in API routes.
- Use SvelteKit's hooks for global API middleware.
- Data comes from Prisma via a Planetscale Mysql database.

SEO and Meta Tags


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syntaxfm/website](https://github.com/syntaxfm/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
