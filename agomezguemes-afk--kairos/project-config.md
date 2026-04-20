---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start          # Start Expo dev server
npm run ios        # Run on iOS simulator
npm run android    # Run on Android emulator
npm run web        # Run on web
```

No test or lint scripts are configured yet.

> **Important:** `react-native-reanimated/plugin` must remain the **last** plugin in `babel.config.js`.

## Architecture

**KAIROS** — a React Native/Expo fitness tracking app ("The Training OS").

### Navigation tree

```
AppNavigator (native stack)
├── WelcomeScreen
├── SetupScreen          ← 4-step onboarding form
└── Dashboard (bottom tabs)
    ├── HomeTab
    ├── BlockLibraryScreen
    ├── ProgressTab
    └── ProfileTab
```

### State management

A single **`TrainingContext`** (`src/context/TrainingContext.tsx`) holds all global state: day entries, streak, weekly stats, and sessions. Consume it with `useTraining()`. Local block/exercise state lives in `BlockLibraryScreen` via `useState`.

Persistence (AsyncStorage) and analytics are **not yet implemented** — both have TODO comments in context.

### Styling

Two styling mechanisms coexist:
- **Design tokens** at `src/theme/tokens.ts` — colors, typography, spacing, radii, shadows, animation presets. Dark mode by default (background `#0A0A0F`, accent gold `#C9A96E`).
- **NativeWind v4** (`nativewind`) is configured but minimally used.

Prefer design tokens (`src/theme/tokens.ts`) over hardcoded values for new UI work.

### Animation

- `react-native-reanimated` (entering/exiting animations, e.g. `FadeInDown`)
- `react-native-reanimated`'s `Animated` API for spring/scale/opacity transitions

Use animation presets from `tokens.ts` (`tokens.animation.spring.*`) for consistency.

### Types

```
src/types/
├── core.ts      ← Discipline, ExerciseCard, WorkoutBlock, Sets
├── training.ts  ← TrainingSession, MuscleGroup, TrainingType
├── progress.ts  ← DayEntry, StreakData, WeeklyLog
├── legacy.ts    ← Deprecated — avoid
└── index.ts     ← Barrel re-exports from core.ts
```

TypeScript strict mode is enabled.

### Performance conventions

- Wrap pure display components in `React.memo()`
- Use `useMemo` for derived/computed values
- Use `useCallback` for event handlers passed as props

Extended Guidelines (Kairos Product Vision)
The following sections describe the long‑term direction. Use them as inspiration, not as a straightjacket. You are encouraged to propose better approaches, new libraries, or entirely different architectural patterns that still honour the core philosophy.

1. Philosophy & User Freedom
Kairos is not a typical fitness app. It is a personal creation space where each user builds their own training ecosystem. No rigid categories, no forced disciplines.

Core principles (non‑negotiable):

- User decides what to track – Every exercise can have custom metrics (weight, reps, distance, pace, RPE, etc.).
- Editing feels fluid – Prefer inline editing over modals.
- Gestures feel natural – Swipe to delete, drag & drop to reorder.
- Feedback is tangible – Animations, haptics, visual cues.
- Premium minimalism – Clean hierarchy, generous spacing, accent gold (#C9A96E).
- AI is helpful, not intrusive – It analyzes, recommends, generates plans; it never interrupts.

What you can change / improve:
You may propose a different visual identity, new interaction paradigms (e.g., haptic‑first, voice‑assisted), or alternative ways to achieve user freedom (e.g., allowing users to create custom UI layouts). As long as the user feels empowered, anything is on the table.

2. Dynamic Data Model (the heart of Kairos)
The current dynamic model uses FieldDefinition, ExerciseCardDynamic, SetDynamic, WorkoutBlock. This is a solid foundation, but you may:

- Replace it with a different NoSQL‑like structure (e.g., using jsonb fields from the start).
- Integrate a local database like realm or watermelonDB for better performance.
- Add versioning, conflict resolution, or offline sync capabilities.

The only requirement: The user must be able to define custom metrics per exercise, and the AI must be able to analyse any numeric field over time.

3. Component Library (Premium UI)
We have started building components like ExerciseCardDynamic, WorkoutBlockDynamic, IconPicker, ColorPicker, NumericSelector. However, you are free to:

- Replace them with more polished libraries (e.g., dripsy, gluestack-ui, or fully custom animated components).
- Introduce new interaction patterns (e.g., long‑press context menus, bottom sheets for quick editing).
- Implement drag & drop using react-native-draggable-flatlist or react-native-gesture-handler + Reanimated.
- Add skeletal loading, shimmer effects, or micro‑interactions.

Goal: The UI should feel top‑tier – something that could be featured by Apple. If you know a better way to achieve that, go for it.

4. State Management & Persistence
Current plan: AsyncStorage + later Zustand + Supabase. You may:

- Use react-native-mmkv for faster offline storage.
- Adopt react-query (TanStack Query) for remote sync.
- Implement expo-sqlite for complex queries.
- Build a custom offline‑first engine.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agomezguemes-afk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
