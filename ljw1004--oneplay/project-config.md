---
trigger: always_on
description: We have two mobile-first single-page PWAs: music/ indexes a user's OneDrive music folder and plays it back with full offline support; video/ indexes and plays video, and supports Chromecast, but has no offline support. Both are framework-free TypeScript with zero runtime dependencies. Targets iOS Safari, with Chromium for development and testing.
---

# Guidance for Project Agents

We have two mobile-first single-page PWAs: music/ indexes a user's OneDrive music folder and plays it back with full offline support; video/ indexes and plays video, and supports Chromecast, but has no offline support. Both are framework-free TypeScript with zero runtime dependencies. Targets iOS Safari, with Chromium for development and testing.

We are working on the music app, unless otherwise stated.

- AGENTS.md -- this current file is always inserted into the start of every agent conversation, and explains (1) how the user wants the AI to behave, (2) development instructions for this repo.
- LEARNINGS.md -- you *MUST ALWAYS* read this file, which contains your hard-won wisdom experience, learned through costly trial and error. You must read this to avoid making the same mistakes in all work you do.
- OnePlay Music app in `music/*`
  - music/ai/ARCHITECTURE.md -- the notes you have created about the architecture of the OnePlay Music app. You *MUST* read this to learn the overall project architecture, structure, components, dataflow, invariants. You are expected to study this before doing any work, so you can fit into existing codebase patterns and structures.
  - music/ai/PLAN*.md -- all other notes you've created while authoring the app.
  - music/ai/DESIGN.md -- the UX spec document for the OnePlay music app.
- OnePlay Video app in `video/*`
  - this has not yet been created!

## Cross-cutting issues

- Everything should typecheck clean
- Offline-first architecture: the app always launches from local data (IndexedDB) and is immediately usable. All network calls are background and non-blocking. The network is assumed unreliable at all times — partial connectivity (unauthenticated WiFi, weak cellular) is the norm, not the exception. This principle applies from M3 onward: the index loads from cache first, the tree renders from cache, playback uses cached audio when available, and network failures are surfaced to the user without blocking the UI.
- Scale-aware: the music library may contain up to 30k tracks in deep directory hierarchies. Every component that touches the index or tree must handle this without blocking the UI thread. The breadcrumb+children design helps (only a small slice is visible at a time), but search results, recursive track counting, and indexing must all be designed with this scale in mind.
- Auth permeates fetch: every network call to OneDrive goes through an authenticated fetch wrapper with automatic token refresh, retry on 429/503, and timeout.

## Codebase style and guidelines

Coding style: All code must also be clean, documented and minimal. That means:
- Keep It Simple Stupid (KISS) by reducing the "Concept Count". That means, strive for fewer
  functions or methods, fewer helpers. If a helper is only called by a single callsite,
  then prefer to inline it into the caller.
- At the same time, Don't Repeat Yourself (DRY)
- There is a tension between KISS and DRY. If you find yourself in a situation where
  you're forced to make a helper method just to avoid repeating yourself, the best
  solution is to look for a way to avoid even having to do the complicated work at all.
- If some code looks heavyweight, perhaps with lots of conditionals, then think harder for a more elegant way of achieving it.
- Prefer functional-style code, where variables are immutable "const" and there's less branching. Prefer to use ternary expressions "b ? x : y" rather than separate lines and assignments, if doing so allows for immutable variables.
- Code should have comments, and functions should have docstrings. The best comments are ones that introduce invariants, or prove that invariants are being upheld, or indicate which invariants the code relies upon.
- **Name side-effecting functions to expose the side effect.** A function named `load()` implies it returns data. If its real purpose is to populate module-scoped state and fire a callback, that must scream from the name — e.g. `loadIntoStateAndNotify()`. Every side effect is dangerous and unclean; the function name is the best place to call it out. Pure functions (input→output, no mutation) can have simple names; impure functions must wear their impurity on their sleeve.
- **Never use `void asyncFn()` for fire-and-forget.** Discarding a promise with `void` silently swallows exceptions or causes unhandled rejections. The caller should always `await` the async function. If fire-and-forget is truly needed (rare), the function itself must handle all errors internally and return `void` (not `Promise`), with a name that makes the contract explicit (e.g. `saveFireAndForget()`). Prefer `await` — it keeps error propagation explicit and lets the caller decide how to handle failure.

I am adamant about clean engineering. What I look for:
- Learnings must be stored in root `LEARNINGS.md`, or in other files linked from it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ljw1004/oneplay](https://github.com/ljw1004/oneplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
