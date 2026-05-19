---
trigger: always_on
description: This is non-negotiable. Every conversation forgets this. **READ THIS EVERY TIME.**
---

# CLAUDE.md — bun-queue

---

## ABSOLUTE RULES — READ BEFORE WRITING ANY CODE

### RULE 1: ZERO VANILLA JAVASCRIPT IN CLIENT CODE

This is non-negotiable. Every conversation forgets this. **READ THIS EVERY TIME.**

If you are about to write `document.`, `window.`, `localStorage.`, or an IIFE — **STOP**. There is a stx composable for it. If there isn't,**add one to stx upstream**. No exceptions. No "just this once." No "it's simpler."

### RULE 2: NO POLLING — USE REACTIVE PATTERNS

Never write `setInterval` + fetch loops, `setTimeout` retry loops, or any polling pattern. Use:

- `useFetch()` with reactive dependencies for data that changes
- `useEventListener('stx:navigate', fn)` for navigation-triggered refreshes
- Server-sent events or WebSocket composables for real-time data (add to stx if missing)

### RULE 3: USE STX DIRECTIVES — NOT JS DOM MANIPULATION

All UI behavior goes through stx directives and the reactive system. No `classList.add()`, no `style.display =`, no `innerHTML =`. Use `:class`, `:style`, `:if`, `@click`, `{{ }}`.

### RULE 4: CHECK STX SOURCE BEFORE GUESSING

Before writing **any** frontend code, read these files to know what's available:

| What | File to read |
|------|-------------|
| Browser runtime API | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/signals.ts` |
| Template directives | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/process.ts` |
| Expression evaluation | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/expressions.ts` |
| Reactive system | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/reactivity.ts` |
| SPA router | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/router/src/client.ts` |
| Browser composables | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/browser-composables.ts` |
| Composable types | `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/composables.ts` |

Search for `window.stx = {` in `signals.ts` to see every function exposed to the browser. Search for `window.` assignments after it to see all globals.

### RULE 5: FIX STX UPSTREAM — NEVER WORK AROUND IT

If stx is missing a composable, directive, or has a bug:

1. Fix it in `/Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx/src/`
2. Take inspiration from Vue 3 Composition API, Svelte 5 runes, React hooks
3. Rebuild: `cd /Users/glennmichaeltorregosa/Documents/Projects/stx/packages/stx && bun --bun build.ts`
4. If router changes: `cd /Users/glennmichaeltorregosa/Documents/Projects/stx/packages/router && bun --bun build.ts`
5. Test in bun-queue devtools: `bun --watch server.ts`

---

## BANNED PATTERNS — EXACT REPLACEMENTS

| Banned pattern | Use instead |
|----------------|-------------|
| `document.getElementById('x')` | `useRef('x')` with `ref="x"` on element |
| `document.querySelector('.x')` | `useRef('x')` or stx directives |
| `document.querySelectorAll('.x')` | `@for` / `:for` directive |
| `document.createElement('div')` | stx templates / `:if` to show/hide |
| `document.addEventListener('click', fn)` | `@click="fn()"` directive |
| `document.addEventListener('submit', fn)` | `@submit.prevent="fn()"` directive |
| `window.addEventListener('resize', fn)` | `useEventListener('resize', fn)` |
| `window.addEventListener('keydown', fn)` | `useEventListener('keydown', fn)` |
| `window.addEventListener('storage', fn)` | `useLocalStorage()` handles cross-tab sync |
| `window.location.href` | `useRoute()` to read, `navigate(url)` to change |
| `window.location.pathname` | `useRoute().path` |
| `window.history.pushState()` | `navigate(url)` |
| `window.history.back()` | `goBack()` |
| `localStorage.getItem(k)` | `useLocalStorage(k, default)` |
| `localStorage.setItem(k, v)` | `useLocalStorage(k, default)` — write via `.set()` |
| `sessionStorage.getItem(k)` | `useSessionStorage(k, default)` (add to stx if missing) |
| `setTimeout(fn, ms)` | `useTimeout(fn, ms)` |
| `setInterval(fn, ms)` | `useInterval(fn, ms)` |
| `setInterval` + fetch (polling) | `useFetch()` with reactive deps or SSE |
| `el.classList.add('x')` | `:class="{ } condition x:"` directive |
| `el.classList.toggle('x')` | `:class="{ } isActive() x:"` with signal |
| `el.style.display = 'none'` | `:if="condition"` directive |
| `el.style.color = 'red'` | `:style="{ color: val() }"` directive |
| `el.innerHTML = '...'` | `{{ expression }}` or `{!! rawHtml !!}` |
| `el.textContent = '...'` | `{{ expression }}` |
| `el.setAttribute('href', x)` | `:href="expression"` |
| `new MutationObserver(...)` | `effect()` — stx tracks reactive deps automatically |
| `(function() { ... })()` | `stx.mount(fn)` or `stx.mountEl(sel, fn)` |
| `fetch().then().then()` in IIFE | `useFetch(url)` or `useAsync(fn)` |
| Custom SPA router | `injectRouterScript()` — canonical router handles everything |

---

## EXCEPTIONS — LEGITIMATE EXTERNAL LIB USAGE

These are the **only** cases where direct DOM/window access is acceptable:

| Exception | Why | Rule |
|-----------|-----|------|
| **Chart.js initialization** | Chart.js requires a canvas context via its own API | Wrap in `onMount()`, destroy in `onDestroy()`, get canvas via `useRef()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacksjs/bun-queue](https://github.com/stacksjs/bun-queue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
