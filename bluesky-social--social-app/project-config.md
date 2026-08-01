---
trigger: always_on
description: This document provides guidance for working effectively in the Bluesky Social app codebase.
---

# CLAUDE.md – Bluesky Social App Development Guide

This document provides guidance for working effectively in the Bluesky Social app codebase.

## Project Overview

Bluesky Social is a cross-platform social media application built with React Native and Expo. It runs on iOS, Android, and Web, connecting to the AT Protocol (atproto) decentralized social network.

**Tech Stack:**

- React 19.1
- React Native 0.81 with Expo 54
- TypeScript 7
- React Navigation 7 for routing
- TanStack Query (React Query) for data fetching
- Lingui 5 for internationalization
- Custom design system called ALF (Application Layout Framework)

Prefer using the latest features available for each of these libraries (exact versions are found in `package.json`). For example, prefer `@lingui/react/macro` over `@lingui/react`. Suggest refactoring legacy or deprecated uses.

## Essential Commands

```bash
# Development
pnpm start              # Start Expo dev server
pnpm web                # Start web version
pnpm android            # Run on Android
pnpm ios                # Run on iOS

# Testing & Quality
# IMPORTANT: Always use these pnpm scripts, never call the underlying tools directly
pnpm test               # Run Jest tests
pnpm lint               # Run Oxlint
pnpm typecheck          # Run TypeScript type checking
pnpm prettier           # Run Prettier for code formatting

# Internationalization
# DO NOT run these commands - extraction and compilation are handled by CI
pnpm intl:extract       # Extract translation strings (nightly CI job)
pnpm intl:compile       # Compile translations for runtime (nightly CI job)

# Build
pnpm build-web          # Build web version
pnpm prebuild           # Generate native projects
```

## Project Structure

```
src/
├── alf/                    # Design system (ALF) - themes, atoms, tokens
├── components/             # Shared UI components (Button, Dialog, Menu, etc.)
├── screens/                # Full-page screen components (newer pattern)
├── features/               # Macro-features that bridge components/screens
├── view/
│   ├── screens/            # Full-page screens (legacy location)
│   ├── com/                # Reusable view components
│   └── shell/              # App shell (navigation bars, tabs)
├── state/
│   ├── queries/            # TanStack Query hooks
│   ├── preferences/        # User preferences (React Context)
│   ├── session/            # Authentication state
│   └── persisted/          # Persistent storage layer
├── lib/                    # Utilities, constants, helpers
├── locale/                 # i18n configuration and language files
└── Navigation.tsx          # Main navigation configuration
```

### Project Structure in Depth

When building new things, follow these guidelines for where to put code.

#### Components vs Screens vs Features

**Components** are reusable UI elements that are not full screens. Should be
platform-agnostic when possible. Examples: Button, Dialog, Menu, TextField. Put
these in `/components` if they are shared across screens.

**Screens** are full-page components that represent a route in the app. They
often contain multiple components and handle layout for a page. New screens
should go in `/screens` (not `/view/screens`) to encourage better organization
and separation from legacy code.

For complex screens that have specific components or data needs that _are not
shared by other screens_, we encourage subdirectories within `/screens/<name>`
e.g. `/screens/ProfileScreen/ProfileScreen.tsx` and
`/screens/ProfileScreen/components/`.

**Features** are higher-level modules that may include context, data fetching,
components, and utilities related to a specific feature e.g.
`/features/liveNow`. They don't neatly fit into components or screens and often
span multiple screens. This is an optional pattern for organizing complex
features.

#### Legacy Directories

For the most part, avoid writing new files into the `/view` directory and
subdirectories. This is the older pattern for organizing screens and components,
and it has become a bit disorganized over time. New development should go into
`/screens`, `/components`, and `/features`.

#### State

The `/state` directory is where we've historically put all our data fetching and
state management logic. This is perfectly fine, but for new features, consider
organizing state logic closer to the components that use it, either within a
feature directory or co-located with a screen. The key is to keep related code
together and avoid having "god files" with too much unrelated logic.

#### Lib

The `/lib` directory is for utilities and helpers that don't fit into other
categories. This can include things like API clients, formatting functions,
constants, and other shared logic.

#### Top Level Directories

Avoid writing new top-level subdirectories within `/src`. We've done this for a
few things in the past that, but we have stronger patterns now. Examples:
`/logger` should probably have been written into `/lib`. And `ageAssurance` is
better classified within `/features`. We will probably migrate these things
eventually.

### File and Directory Naming Conventions

Typically JS style for variables, functions, etc. We use ProudCamelCase for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluesky-social/social-app](https://github.com/bluesky-social/social-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
