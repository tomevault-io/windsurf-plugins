---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.

Only delegate to subagents when the task clearly benefits from a separate agent with a new context window.

<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>

ALWAYS read and understand relevant files before proposing code edits. Do not speculate about code you have not inspected. If the user references a specific file/path, you MUST open and inspect it before explaining or proposing fixes. Be rigorous and persistent in searching code for key facts. Thoroughly review the style, conventions, and abstractions of the codebase before implementing new features or abstractions.

<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>

Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable.

## Project Overview

**motion-start** is a Svelte 5 animation library inspired by framer-motion (React). It provides motion components (`Motion.div`, `Motion.button`, etc.), AnimatePresence for exit animations, layout animations, and gesture handling. Currently in alpha and actively being migrated to Svelte 5 with runes.

## Commands

```bash
# Development
bun dev                     # Start dev server on localhost:5000

# Build
bun run build              # Build the library (svelte-package + publint)
bun run package            # Sync and package the library

# Testing
bun test                   # Run Vitest unit tests
bun test <pattern>         # Run specific tests matching pattern
bun run test:ui            # Open Vitest UI
bun run cypress            # Open Cypress GUI
bun run cypress:run        # Run all Cypress specs in 2 local shards with videos
bun run cypress:run:serial -- --spec "cypress/integration/<name>.ts"  # With bun run dev already running

# Type checking and linting
npx sv check               # Svelte type checking (run before commits)
bun run lint               # Biome linting
npx @biomejs/biome format --write .  # Format code
```

## Architecture

### Core Library (`src/lib/motion-start/`)

The library mirrors framer-motion's architecture adapted for Svelte 5:

- **motion/**: Core `Motion.svelte` component and `createRendererMotionComponent` factory. Components are created via proxies (`motion.div`, `m.div`).
- **render/**: VisualElement system - the abstraction layer between components and DOM/SVG rendering. `VisualElement.svelte.ts` manages animations outside React's render cycle.
- **components/**: Higher-order components:
  - `AnimatePresence/` - Manages exit animations for unmounting children
  - `LayoutGroup/` - Groups layout animations
  - `Reorder/` - Drag-to-reorder functionality
  - `LazyMotion/` - Code-splitting for features
- **context/**: Svelte 5 context system using `$state` for reactivity. Key contexts: `PresenceContext`, `MotionConfigContext`, `LayoutGroupContext`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JonathonRP/motion-start](https://github.com/JonathonRP/motion-start) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
