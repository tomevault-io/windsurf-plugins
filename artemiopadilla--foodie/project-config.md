---
trigger: always_on
description: **Project**: Foodie - Comprehensive Meal Planning Web Application
---

# CLAUDE.md - Foodie Meal Planning PWA

**Project**: Foodie - Comprehensive Meal Planning Web Application
**Version**: 1.0.0
**Last Updated**: 2025-01-10
**Tech Stack**: React 18, TypeScript, Vite 5, Tailwind CSS, Firebase (optional), GitHub Pages

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Tech Stack Decisions](#tech-stack-decisions)
4. [Project Structure](#project-structure)
5. [Key Features](#key-features)
6. [Data Model](#data-model)
7. [State Management](#state-management)
8. [Internationalization](#internationalization)
9. [PWA Implementation](#pwa-implementation)
10. [GitHub Integration](#github-integration)
11. [Development Workflow](#development-workflow)
12. [Deployment](#deployment)
13. [Testing Strategy](#testing-strategy)
14. [Performance Optimization](#performance-optimization)
15. [Accessibility](#accessibility)
16. [Future Enhancements](#future-enhancements)
17. [Troubleshooting](#troubleshooting)
18. [Contributing](#contributing)

---

## Project Overview

Foodie is a Progressive Web Application (PWA) designed to help users plan meals, discover recipes, generate shopping lists, and manage their pantry. The app supports multiple languages (English, Spanish, French) and multiple cuisines, making it truly global.

### Core Objectives

- **Flexibility**: Support any cuisine, dietary restriction, or meal planning style
- **Accessibility**: WCAG 2.1 AA compliant, works offline, mobile-first
- **Community-Driven**: GitHub-integrated recipe contribution system
- **Performant**: Fast loading, optimized bundles, PWA capabilities
- **Multilingual**: Full i18n support with easy expansion to more languages

---

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    User Interface (React)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │  Pages   │  │Components│  │ Layouts  │  │ Routing ││
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘│
└───────┼────────────┼──────────────┼──────────────┼─────┘
        │            │              │              │
┌───────▼────────────▼──────────────▼──────────────▼─────┐
│              Context API (State Management)             │
│  ┌──────┐ ┌──────┐ ┌─────────┐ ┌──────┐ ┌───────────┐ │
│  │Recipe│ │Planner│ │Shopping│ │ Auth │ │ Language  │ │
│  └──────┘ └──────┘ └─────────┘ └──────┘ └───────────┘ │
└────────┬────────────────────────────────────────┬──────┘
         │                                        │
┌────────▼────────────────────────────────────────▼──────┐
│                     Services Layer                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌───────┐│
│  │ Recipe   │  │Firebase  │  │  GitHub   │  │ Utils ││
│  │ Service  │  │  Service │  │   API     │  │       ││
│  └──────────┘  └──────────┘  └───────────┘  └───────┘│
└────────┬────────────────────────────────────────┬──────┘
         │                                        │
┌────────▼────────────────────────────────────────▼──────┐
│                      Data Layer                         │
│  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐│
│  │JSON Files│  │ LocalStorage │  │ Firebase/Firestore││
│  │(Static)  │  │  (Offline)   │  │  (Cloud Sync)     ││
│  └──────────┘  └──────────────┘  └──────────────────┘│
└─────────────────────────────────────────────────────────┘
```

### Design Patterns

1. **Context + Hooks Pattern**: Global state managed through React Context API with custom hooks
2. **Container/Presentational**: Separation of logic (containers) and UI (presentational components)
3. **Composition**: Small, reusable components composed to build complex UIs
4. **Service Layer**: Business logic abstracted into service modules
5. **Progressive Enhancement**: Core functionality works without JavaScript, enhanced with React

---

## Tech Stack Decisions

### Why These Technologies?

#### React 18 + TypeScript
- **React**: Industry standard, excellent ecosystem, great performance with concurrent features
- **TypeScript**: Type safety reduces bugs, better IDE support, self-documenting code
- **Hooks**: Modern React patterns, easier state management and side effects

#### Vite 5
- **Fast HMR**: Lightning-fast hot module replacement for better DX
- **ES Modules**: Native ES module support, no bundling in dev
- **Optimized Build**: Rollup-based production builds with automatic code splitting
- **Plugin Ecosystem**: Easy PWA integration with vite-plugin-pwa

#### Tailwind CSS
- **Utility-First**: Rapid development with utility classes
- **Customization**: Easy to customize theme and extend
- **Performance**: PurgeCSS removes unused styles automatically
- **Dark Mode**: Built-in dark mode support with `class` strategy

#### React Context API (vs Redux/Zustand)
- **Built-in**: No external dependencies for state management
- **Sufficient**: App complexity doesn't require heavy state management
- **Simple**: Easier for contributors to understand and extend
- **Performant**: With proper memoization and context splitting

#### i18next
- **Mature**: Battle-tested i18n solution
- **React Integration**: Official react-i18next package
- **Language Detection**: Automatic language detection from browser

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtemioPadilla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
