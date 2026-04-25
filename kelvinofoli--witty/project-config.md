---
trigger: always_on
description: > **Last Updated**: 2026-01-09
---

# CLAUDE.md - Witty Codebase Guide for AI Assistants

> **Last Updated**: 2026-01-09
> **Project**: Witty - Interactive Grocery List App
> **Tech Stack**: React Native + Expo + TypeScript

## Table of Contents

1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Development Workflows](#development-workflows)
4. [Architecture & Patterns](#architecture--patterns)
5. [Key Conventions](#key-conventions)
6. [AI Assistant Guidelines](#ai-assistant-guidelines)
7. [Common Tasks](#common-tasks)
8. [Testing & Quality](#testing--quality)

---

## Project Overview

**Witty** is a React Native mobile application for managing grocery lists with an interactive canvas-based visualization. The app provides:

- **Expandable grocery list cards** with smooth animations
- **Interactive canvas** using Skia for rendering grocery items with pan/zoom gestures
- **Canvas minimap** for spatial navigation
- **Animated previews** of top items when cards are collapsed
- **Cross-platform support** for iOS, Android, and Web

### Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **React Native** | 0.76.9 | Mobile framework |
| **React** | 18.3.1 | UI library |
| **Expo** | ~52.0.46 | Development platform |
| **TypeScript** | ^5.3.3 | Type safety |
| **React Native Reanimated** | ~3.16.1 | Advanced animations |
| **React Native Skia** | ~1.12.4 | Canvas rendering |
| **React Native Gesture Handler** | ~2.20.2 | Gesture detection |
| **React Navigation** | ~7.0.14 | Navigation (native stack) |

### Project Metadata

- **Bundle ID (iOS)**: `com.kelvinportuphy.witty`
- **Package (Android)**: `com.kelvinportuphy.witty`
- **New Architecture**: Enabled (`newArchEnabled: true`)
- **Typed Routes**: Enabled (experimental)

---

## Codebase Structure

### Directory Layout

```
/home/user/witty/
├── android/                    # Android native code
├── ios/                        # iOS native code
├── scripts/                    # Build and setup scripts
│   └── reset-project.js       # Reset to blank project
├── src/                       # Main source code
│   ├── assets/                # Static assets
│   │   ├── fonts/            # Custom fonts (SpaceMono)
│   │   └── images/           # Images & icons
│   │       └── groceries/    # 10 grocery item images
│   ├── components/            # Reusable components
│   │   └── buttons/
│   │       └── pinButtons.tsx # (Empty placeholder)
│   ├── constants/             # App-wide constants
│   │   └── Colors.ts         # Color definitions (light/dark mode)
│   ├── projects/              # Feature modules (project-based organization)
│   │   └── groceryList/      # Grocery list feature
│   │       ├── components/   # Feature-specific components
│   │       │   ├── canvas.tsx      # Interactive Skia canvas
│   │       │   ├── canvasMap.tsx   # Canvas minimap
│   │       │   ├── header.tsx      # Header component
│   │       │   ├── listItem.tsx    # Expandable list card
│   │       │   └── preview.tsx     # Animated item preview
│   │       ├── data.ts       # Static grocery data
│   │       ├── types.ts      # TypeScript type definitions
│   │       └── index.tsx     # Main grocery list screen
│   ├── routes/                # Navigation configuration
│   │   └── index.tsx         # React Navigation setup
│   └── utils/                 # Utility functions
│       ├── index.ts          # Helper functions (e.g., degressToRadians)
│       └── styles/
│           └── index.ts      # Shared style constants (rowStyle)
├── App.tsx                    # Root component
├── index.js                   # Expo entry point
├── app.json                   # Expo configuration
├── package.json               # Dependencies & scripts
└── tsconfig.json              # TypeScript configuration
```

### Key Files

| File | Purpose |
|------|---------|
| `App.tsx` | Root component that renders RootNavigator |
| `src/routes/index.tsx` | Navigation setup (React Navigation Native Stack) |
| `src/projects/groceryList/index.tsx` | Main grocery list screen with FlatList |
| `src/projects/groceryList/components/listItem.tsx` | Core component: expandable grocery card |
| `src/projects/groceryList/components/canvas.tsx` | Interactive Skia canvas with pan gestures |
| `src/projects/groceryList/data.ts` | Static grocery list data |
| `src/projects/groceryList/types.ts` | TypeScript types for ListItem & Item |
| `src/constants/Colors.ts` | Light/dark mode color definitions |

---

## Development Workflows

### Setup & Installation

```bash
# Install dependencies
npm install

# Start development server
npm start
# or
npx expo start

# Run on specific platforms
npm run android      # Android emulator
npm run ios          # iOS simulator
npm run web          # Web browser
```

### Available Scripts

| Command | Description |
|---------|-------------|
| `npm start` | Start Expo development server |
| `npm run android` | Run on Android emulator |
| `npm run ios` | Run on iOS simulator |
| `npm run web` | Run in web browser |
| `npm test` | Run Jest tests in watch mode |
| `npm run lint` | Run Expo linter |
| `npm run reset-project` | Reset to blank project template |

### Development Server

- The app uses **Expo Go** or **development builds** for testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kelvinOfoli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
