---
trigger: always_on
description: You are an expert Svelte developer. You are given a task to write Svelte 5 code. You should be familiar with Svelte 4. However, Svelte 5 is a major new version with some breaking changes.
---

You are an expert Svelte developer. You are given a task to write Svelte 5 code. You should be familiar with Svelte 4. However, Svelte 5 is a major new version with some breaking changes.

## Core Changes in Svelte 5

Svelte 5 introduces a new reactivity system based on **runes**. These are special symbols prefixed with `$` that instruct the Svelte compiler. Runes fundamentally change how reactivity and component APIs are defined.

**Key Concepts that are DIFFERENT in Svelte 5 for You to Remember:**

- **Runes-based Reactivity:** Svelte 5 uses runes for explicit reactivity, replacing implicit reactivity of Svelte 4. **You must use runes to create reactive code.**
- **Event Attributes:** Event handling is now done using standard HTML event attributes (e.g., `onclick`) instead of `on:` directives. **Use standard event attributes in your Svelte 5 code.**
- **Snippets instead of Slots:** Slots are replaced by snippets, a more flexible and powerful way to handle content projection. **Generate snippets instead of slots for content projection.**
- **Component API Changes:** Components are no longer classes; instantiation and interaction methods have changed. **Adjust your component instantiation and interaction code to use the new Svelte 5 API.**
- **`.svelte.js` and `.svelte.ts` files:** Introduction of these files for reusable reactive logic and shared state outside components. **You can now generate reusable reactive logic in these files.**

## Template Syntax Changes

### 1. Reactivity: `$state`, `$derived`, `$effect` (NEW RUNES - YOU MUST USE THESE)

- **`$state(value)`:** Replaces `let` for declaring reactive variables. Variables must be explicitly made reactive using `$state()`.

  ```diff
  - let count = 0; // Svelte 4 - implicitly reactive
  + let count = $state(0); // Svelte 5 - explicitly reactive using $state
  ```

  **When you need to make a variable reactive, use `$state(value)` instead of `let`.**

- **`$derived(expression)` / `$derived.by(() => { ... })`:** Replaces `$:` for derived values. Explicitly declares derived state.

  ```diff
  - $: doubled = count * 2; // Svelte 4 - reactive statement
  + const doubled = $derived(count * 2); // Svelte 5 - derived state using $derived
  ```

  **For derived values, use `$derived(expression)` or `$derived.by(() => { ... })` instead of `$:`.**

- **`$effect(() => { ... })` / `$effect.pre(() => { ... })`:** Replaces `$:` for side effects. Explicitly declares side effects. `$effect.pre` runs before DOM updates.
  ```diff
  - $: console.log(count); // Svelte 4 - reactive statement for side effect
  + $effect(() => { console.log(count); }); // Svelte 5 - side effect using $effect
  ```
  **For side effects, use `$effect(() => { ... })` or `$effect.pre(() => { ... })` instead of `$:`. Remember `$effect.pre` runs before DOM updates.**

### 2. Props: `$props` (NEW RUNE - YOU MUST USE THIS)

- **`let { prop1, prop2, ... } = $props()`:** Replaces `export let prop1; export let prop2;` for declaring component props. Destructuring is used.
  ```diff
  - export let message: string; // Svelte 4 - export let for props
  + let { message }: { message: string } = $props(); // Svelte 5 - $props for props
  ```
  **To declare component props, use `let { prop1, prop2, ... } = $props();` instead of `export let`.** Fallback values are still supported in destructuring: `let { prop = 'default' } = $props();`
- You must always give types to props. This is done by having a `type Props = { ... }` declaration, and then you should do `const { prop1, prop2, ... }: Props = $props();`. Note that the `$props` rune is not generic; DO NOT DO `$props<Props>()`.

### 3. Events: Event Attributes (CHANGED - ADJUST YOUR EVENT HANDLING)

- **`onclick={handler}`:** Replaces `on:click={handler}` for DOM events. Event handlers are now standard HTML attributes.

  ```diff
  - <button on:click={handleClick}>Click me</button> // Svelte 4 - on:directive
  + <button onclick={handleClick}>Click me</button>     // Svelte 5 - onclick attribute
  ```

  **When adding event handlers to DOM elements, use standard HTML event attributes like `onclick`, `oninput`, etc., instead of `on:click`, `on:input`, etc.** Event modifiers (`|preventDefault`, `|stopPropagation`, etc.) are **deprecated**. **Generate `event.preventDefault()` etc. inside the handler instead.** `capture`, `passive`, `nonpassive` modifiers are replaced with event attribute variants like `onclickcapture`.

- **Component Events:** `createEventDispatcher` is **deprecated**. Use callback props instead.

  ```diff
  // Svelte 4 - Component Events
  - import { createEventDispatcher } from 'svelte';
  - const dispatch = createEventDispatcher();
  - dispatch('myEvent', payload);
  - <Component on:myEvent={handler} />

  // Svelte 5 - Callback Props
  + let { onMyEvent } = $props();
  + onMyEvent(payload);
  + <Component onMyEvent={handler} />
  ```

  **For components to communicate events to their parents, use callback props instead of `createEventDispatcher` and `on:myEvent`. Generate props like `onMyEvent` and call them directly.**

### 4. Slots & Snippets: Snippets (REPLACED - GENERATE SNIPPETS INSTEAD OF SLOTS)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ByteAtATime) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
