---
trigger: always_on
description: - [Project Overview](#project-overview)
---

# AGENTS.md - Fast Expo App Monorepo

## Table of Contents

- [Project Overview](#project-overview)
- [Monorepo Architecture](#monorepo-architecture)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Development Workflows](#development-workflows)
- [Turborepo & Workspaces](#turborepo--workspaces)
- [CLI Package](#cli-package)
- [Template (Boilerplate)](#template-boilerplate)
- [Website Package](#website-package)
- [Optional Modules](#optional-modules)
- [Coding Conventions](#coding-conventions)
- [Scripts Reference](#scripts-reference)
- [Best Practices](#best-practices)

---

## Project Overview

**Fast Expo App** is a complete monorepo containing:

1. A **CLI tool** (`fast-expo-app`) for scaffolding React Native projects
2. A **production-ready boilerplate** with modern best practices
3. A **landing website** to showcase the project

### What's Included (v3.0.0)

#### Core Stack (Always Included)

✅ **Expo SDK 54** with New Architecture enabled  
✅ **React Native 0.81** with Fabric renderer  
✅ **TypeScript 5.9** with strict mode  
✅ **Expo Router v6** for file-based routing with typed routes  
✅ **MMKV v4** for ultra-fast storage (Nitro Modules, ~30x faster)  
✅ **expo-haptics** for native haptic feedback  
✅ **react-native-edge-to-edge** for modern edge-to-edge display  
✅ **react-native-worklets** for high-performance animations

#### Styling Options (Choose One)

✅ **NativeWind v4** - Tailwind CSS v3 for React Native with dark mode  
✅ **Unistyles v3** - Type-safe styling with 3-theme system (light/dark/premium)  
✅ **Uniwind v1.2** - Tailwind CSS v4 with live theme switching (light/dark/premium)

#### Optional Modules

✅ **TanStack Query v5** - Data fetching with MMKV persistence  
✅ **Zustand** - State management with MMKV storage  
✅ **Expo Dev Client** - Enhanced debugging  
✅ **Jest** - Unit testing

#### Pre-configured

✅ **Theme System** - Persistent themes with MMKV  
✅ **ESLint + Prettier** - Code quality tools  
✅ **Absolute Imports** - `@/` prefix for clean imports  
✅ **Turborepo** - Monorepo management

---

## Monorepo Architecture

This project follows the **create-expo-stack** architecture pattern with CLI source at the root level.

### Key Design Decisions

1. **CLI source** in `/cli/` (not in packages) for easier development
2. **Templates** in `/cli/templates/` for better organization
3. **Modules** in `/cli/modules/` for future extensibility
4. **Published package** in `/packages/fast-expo-app/`
5. **Website** in `/www/` for marketing and documentation

### Benefits

- 🎯 **Clean Separation** - CLI, template, and website are isolated
- ⚡ **Fast Development** - Turborepo caching and parallel execution
- 📦 **Easy Publishing** - Single package ready for npm
- 🔧 **Extensible** - Easy to add new modules and features
- 🌐 **Production Ready** - Includes landing page and documentation

---

## Project Structure

```
fast-expo-app-monorepo/
│
├── cli/                          # 🛠️ CLI Source (root level)
│   ├── src/
│   │   ├── index.ts              # CLI implementation
│   │   ├── generate-project.ts   # Project generation logic
│   │   ├── module-manager.ts     # Module management
│   │   └── ascii-art.ts          # CLI banner
│   ├── templates/
│   │   └── base/                 # 📱 React Native Template
│   │       ├── app/              # Expo Router routes
│   │       ├── components/       # React components
│   │       ├── lib/              # Core utilities
│   │       ├── providers/        # React providers
│   │       ├── constants/        # App constants
│   │       ├── assets/           # Images, fonts
│   │       ├── android/          # Android native
│   │       ├── ios/              # iOS native
│   │       ├── __tests__/        # Jest tests (optional)
│   │       └── README.md
│   ├── modules/                  # 🔮 Conditional modules
│   │   ├── app/                  # Screen modules (styling-specific)
│   │   ├── components/           # Component modules
│   │   ├── constants/            # Constants (themes, colors)
│   │   ├── layout/               # Layout modules (styling-specific)
│   │   ├── lib/                  # Library modules (hooks, storage)
│   │   ├── query-provider/       # React Query providers
│   │   ├── storage/              # MMKV storage modules
│   │   ├── styling/              # Styling configs (Unistyles)
│   │   └── utils/                # Utility modules (colors)
│   ├── tsconfig.json             # CLI TypeScript config
│   └── README.md                 # CLI documentation
│
├── packages/
│   └── fast-expo-app/            # 📦 Published NPM Package
│       ├── bin/
│       │   └── fast-expo-app.js  # Executable entry point
│       ├── dist/                 # Compiled code (from /cli/src/)
│       │   ├── index.js
│       │   ├── index.d.ts
│       │   └── *.map
│       ├── package.json          # NPM package config
│       ├── README.md             # Package documentation
│       └── CHANGELOG.md          # Version history
│
├── www/                          # 🌐 Landing Website (Next.js)
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── global.css
│   ├── components/
│   │   ├── magicui/              # UI components
│   │   ├── AnimatedBeams.tsx
│   │   ├── OrbitingCircleDemo.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Teczer/fast-expo-app](https://github.com/Teczer/fast-expo-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
