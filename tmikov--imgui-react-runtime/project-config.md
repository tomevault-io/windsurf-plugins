---
trigger: always_on
description: **⚠️ CRITICAL: This document (CLAUDE.md) must be kept up-to-date whenever you make changes to the project.**
---

# React Reconciler for DearImGUI

## IMPORTANT: Keeping This Document Updated

**⚠️ CRITICAL: This document (CLAUDE.md) must be kept up-to-date whenever you make changes to the project.**

When modifying the project structure, build system, architecture, or implementation details, you MUST update the relevant sections of this document immediately. This ensures:
- Accurate context for future development sessions
- Proper documentation of architectural decisions
- Clear understanding of how the system works

Always update CLAUDE.md as part of your work - do not defer documentation updates.

## IMPORTANT: Development Workflow

**⚠️ DO NOT COMMIT CODE BEFORE USER REVIEW AND TESTING**

When implementing changes:
1. Write and test the code
2. Wait for the user to review the changes
3. Wait for the user to test the implementation
4. Only commit after explicit user approval

Never commit code immediately after implementation. The user must review and test first.

**⚠️ COMMIT MESSAGE STYLE**

Write commit messages that are factual and technical:
- State what was changed, not how you feel about it
- Use objective language without emotion, opinions, or exaggeration
- Avoid words like "awesome", "amazing", "great", "excellent", "beautiful"
- Be concise and descriptive
- Focus on the technical change and its purpose
- Wrap lines at 72 characters

Good examples:
- "Add ICU library linking for Linux compatibility"
- "Fix case-sensitive filesystem issue in Hermes includes"
- "Pass compiler settings to Hermes external project"

Bad examples:
- "Amazing fix for the awesome Linux build!"
- "Greatly improve the build system"
- "Make things work better"

## Project Overview

This project implements a custom React reconciler that renders to DearImGUI using Static Hermes. The goal is to use React's declarative component model and JSX syntax to describe ImGUI interfaces, while learning how React works internally.

## Implementation

Full ImGUI integration using Static Hermes FFI:
- Complete C++/JavaScript bridge with zero-overhead FFI
- Real-time rendering with Sokol + DearImGUI
- Auto-generated ImGUI bindings (500KB+ of FFI declarations)
- Compiled to native code for maximum performance
- See [llm.md](llm.md) for detailed architecture documentation

## Project Structure

### Directory Layout

```
guireact/
├── lib/                          # Reusable library code
│   ├── jslib-unit/              # Event loop and runtime polyfills
│   ├── imgui-unit/              # ImGui FFI bindings and renderer
│   ├── imgui-runtime/           # C++ runtime infrastructure
│   └── react-imgui-reconciler/  # Custom React reconciler
├── examples/                    # Example applications
│   └── showcase/                # Main showcase application
│       ├── *.jsx                # React application components
│       ├── index.js             # React app entry point
│       └── showcase.cpp         # C++ application entry point
└── external/                    # Third-party native libraries

```

### Component Overview

**Library Components (lib/):**
- **jslib-unit**: Event loop, timers, console polyfills (compiled once)
- **imgui-unit**: FFI bindings, renderer (compiled once)
- **imgui-runtime**: C++ runtime, Hermes integration, Sokol lifecycle
- **react-imgui-reconciler**: Custom React reconciler (bundled with app)

**Example Applications (examples/):**
- **showcase/**: Main showcase application
  - React application components (JSX files)
  - **showcase.cpp**: Application entry point

## Three-Unit Architecture (jslib + React + ImGUI)

We've implemented a three-unit architecture to work around Static Hermes's typed/untyped code separation and provide a proper event loop:

1. ✅ React reconciler building component tree
2. ✅ ImGUI demo with Static Hermes FFI
3. ✅ Three-unit architecture with jslib, React, and ImGUI separated
4. ✅ Event loop with setTimeout/setImmediate and microtask support
5. ✅ Application compiles, links, and runs
6. ✅ **Rendering is functional! React components rendering to ImGui successfully**

## Three-Unit Architecture

### The Challenge
Static Hermes has two compilation modes that cannot be mixed in a single unit:
- **Untyped mode**: Standard JavaScript (React, react-reconciler, our app code)
- **Typed mode**: Type-annotated code with zero-cost FFI (required for ImGUI bindings)

### The Solution: Separate Compilation Units

We split the implementation into three separate units that communicate via `globalThis`:

#### **Unit 1: jslib Unit (Untyped)**
**Location:** `lib/jslib-unit/`

**Contains:**
- Event loop implementation (setTimeout, setImmediate, clearTimeout, clearImmediate)
- Task queue management with deadline-based scheduling
- Helper functions for C++ integration (peek, run)
- Console polyfills (console.log → print)
- Process environment polyfills (process.env.NODE_ENV = 'production')

**Build Process:**
- Built in lib/jslib-unit/ with its own CMakeLists.txt
- shermes compiles directly → `jslib-unit.o` (untyped, `-Xes6-block-scoping`)
- Compiled once, reused across builds

**What it does:**
- Provides browser-like timer APIs (setTimeout, setImmediate)
- Manages macrotask queue sorted by deadline
- Exposes helper functions to C++ for event loop integration:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmikov/imgui-react-runtime](https://github.com/tmikov/imgui-react-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
