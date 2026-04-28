---
trigger: always_on
description: This document provides instructions for AI coding agents working on the MacroPhoto project. Follow these guidelines to effectively contribute to the codebase.
---

# AI Agent Instructions

This document provides instructions for AI coding agents working on the MacroPhoto project. Follow these guidelines to effectively contribute to the codebase.

## Project Overview

MacroPhoto is a mobile app (React Native + Expo) that uses AI (Claude or GPT-5) to analyze food photos and provide macro nutrient breakdowns. The project uses a **task-based development model** where each task in `TASKS.md` is designed to be completed independently.

## Architecture Snapshot (read first)

- Expo Router app with screens in `/app` and shared UI in `/components`.
- State: Zustand stores (e.g., analysis, settings) under `/lib`.
- AI: `services/ai.ts` uses Vercel AI SDK for Claude/OpenAI; environment keys loaded via `.env.local` and `expo-constants`.
- Media: `hooks/useCamera` + `expo-camera`, `hooks/useImagePicker` for library; tests can use a bundled sample image when `APP_ENV=test`.
- Styling: NativeWind/Tailwind classes; constants in `/constants/theme`.
- Tests: Jest for unit, Maestro for E2E; E2E flows live in `.maestro`.

## Before Starting Any Work

### 1. Read the Core Documentation

Always start by reading these files:

- `SPEC.md` - Complete product specification and requirements
- `TASKS.md` - All development tasks and their status
- This file (`AGENTS.md`) - Agent instructions

### 2. Understand the Current State

- Check the git status to see what files exist
- Look at recently modified files
- Understand what has been completed vs. what's pending

### 3. Identify Your Task

- Find the specific task you're working on in `TASKS.md`
- Ensure you understand the task's:
  - Description
  - Dependencies (complete those first if needed)
  - Acceptance criteria (how to know you're done)
  - Files to create/modify

## Task Selection Guidelines

### How to Pick a Task

1. **Check Dependencies**: Only work on tasks whose dependencies are completed
2. **Start Small**: If new to the project, start with smaller, UI-focused tasks
3. **One Task at a Time**: Complete one full task before moving to the next
4. **Follow the Order**: Tasks are generally ordered by logical flow

### Task Status Updates

**IMPORTANT**: Keep `TASKS.md` clean and focused on pending work only.
All active and future tasks live in `TASKS.md`; remove completed tasks entirely instead of marking them done.

When you start a task:

```markdown
### [~] TASK-XXX: Task name
```

When you complete a task:

1. **Remove it entirely from TASKS.md** - don't mark as [x], just delete the entire task section
2. This keeps the file focused only on what needs to be done
3. Avoids context pollution for the next agent

**Why remove completed tasks?**

- Agents pick up tasks from TASKS.md
- Completed tasks clutter the file and waste tokens
- Fresh agents should see only what's pending, not what's done
- Git history preserves what was completed if needed

## Development Guidelines

### Code Style

1. **TypeScript**: All code must be TypeScript (`.ts`, `.tsx`)
2. **Functional Components**: Use functional components with hooks
3. **NativeWind/Tailwind**: Use Tailwind classes for styling (once configured)
4. **Naming Conventions**:
   - Components: `PascalCase` (e.g., `Button.tsx`, `CameraView.tsx`)
   - Utilities: `camelCase` (e.g., `imageUtils.ts`, `storage.ts`)
   - Constants: `UPPER_SNAKE_CASE` for values, `camelCase` for files
   - Types/Interfaces: `PascalCase` (e.g., `MacroBreakdown`, `AIProvider`)

5. **File Organization**:
   ```
   /app          - Expo Router pages
   /components   - Reusable UI components
   /lib          - Utilities and helpers
   /services     - API clients and external services
   /hooks        - Custom React hooks
   /types        - TypeScript definitions
   /constants    - App configuration and constants
   /assets       - Images, fonts, etc.
   ```

### Component Structure

Follow this pattern for components:

```typescript
// components/Button.tsx
import { TouchableOpacity, Text, ActivityIndicator } from 'react-native';
import { ReactNode } from 'react';

interface ButtonProps {
  onPress: () => void;
  children: ReactNode;
  variant?: 'primary' | 'secondary';
  loading?: boolean;
  disabled?: boolean;
}

export function Button({
  onPress,
  children,
  variant = 'primary',
  loading = false,
  disabled = false
}: ButtonProps) {
  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={disabled || loading}
      className={`p-4 rounded-lg ${
        variant === 'primary' ? 'bg-blue-600' : 'bg-gray-600'
      }`}
    >
      {loading ? (
        <ActivityIndicator color="white" />
      ) : (
        <Text className="text-white font-semibold text-center">
          {children}
        </Text>
      )}
    </TouchableOpacity>
  );
}
```

### TypeScript Types

Define types in `/types`:

```typescript
// types/nutrition.ts
export interface MacroBreakdown {
  mealName: string;
  calories: number;
  proteinG: number;
  carbsG: number;
  fatG: number;
  confidence: 'low' | 'medium' | 'high';
  notes?: string;
}

export type AIProvider = 'claude' | 'gpt5';

export interface AppSettings {
  aiProvider: AIProvider;
  claudeApiKey?: string;
  openaiApiKey?: string;
}
```

### API Integration Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gricha/macro-photo](https://github.com/gricha/macro-photo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
