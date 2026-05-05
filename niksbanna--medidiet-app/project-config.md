---
trigger: always_on
description: MediDiet AI is a React Native mobile application built with Expo that provides AI-powered personalized nutrition management for individuals with medical conditions such as diabetes, hypertension, kidney disease, and other health conditions.
---

# GitHub Copilot Instructions for MediDiet AI

## Project Overview

MediDiet AI is a React Native mobile application built with Expo that provides AI-powered personalized nutrition management for individuals with medical conditions such as diabetes, hypertension, kidney disease, and other health conditions.

### Purpose
- Generate personalized weekly meal plans using Google Gemini AI
- Track daily meal intake and adherence to dietary plans
- Provide medical-condition-specific nutritional guidance
- Monitor health metrics and progress over time

## Tech Stack

### Core Technologies
- **React Native**: 0.79.3
- **Expo**: ~53.0.9 with Expo Router for navigation
- **TypeScript**: 5.8.3 (strict mode enabled)
- **Node.js**: v18 or higher

### Key Libraries
- **expo-router**: File-based routing system
- **@react-native-async-storage/async-storage**: Local data persistence
- **expo-linear-gradient**: UI gradients
- **@expo/vector-icons (MaterialIcons)**: Icon system
- **react-native-safe-area-context**: Safe area handling

### AI Integration
- **Google Gemini API**: AI-powered meal plan generation via `gemini-2.0-flash-exp` model
- **Fallback System**: Local meal generation when API is unavailable

## Project Structure

```
medidiet-app/
├── app/                    # Expo Router screens
│   ├── (tabs)/            # Tab-based navigation screens
│   │   ├── index.tsx      # Dashboard/Home screen
│   │   ├── plan.tsx       # Meal plan screen
│   │   ├── log.tsx        # Meal logging screen
│   │   ├── profile.tsx    # User profile screen
│   │   └── _layout.tsx    # Tab layout configuration
│   ├── onboarding.tsx     # Multi-step onboarding flow
│   ├── index.tsx          # Root entry screen
│   └── _layout.tsx        # Root layout
├── components/
│   └── ui/                # Reusable UI components
├── contexts/              # React Context providers
│   └── HealthContext.tsx  # Global health state management
├── hooks/                 # Custom React hooks
│   └── useHealth.tsx      # Health context hook
├── services/              # Business logic and API services
│   ├── aiDietService.ts   # Diet plan generation with fallback
│   └── geminiService.ts   # Google Gemini API integration
├── types/                 # TypeScript type definitions
│   └── health.ts          # Health-related interfaces
├── utils/                 # Utility functions
│   └── toast.ts           # Toast notification helpers
└── constants/             # App-wide constants
    └── Colors.ts          # Color scheme definitions
```

## Coding Standards

### TypeScript
- **Always use TypeScript**: No plain JavaScript files (.js)
- **Strict mode enabled**: All TypeScript strict checks are active
- **Type safety**: Always provide explicit types for function parameters and return values
- **Interface over type**: Prefer `interface` for object shapes, use `type` for unions/intersections
- **No `any` type**: Use proper types or `unknown` if type is truly unknown

### React Native / React Conventions
- **Functional Components**: Use function components with hooks, no class components
- **Hooks**: Follow React hooks rules (useEffect, useState, useContext, custom hooks)
- **File naming**: 
  - Components: PascalCase (e.g., `MealPlanScreen.tsx`)
  - Hooks: camelCase with `use` prefix (e.g., `useHealth.tsx`)
  - Services: camelCase (e.g., `aiDietService.ts`)
  - Types: camelCase (e.g., `health.ts`)
- **Component structure**:
  ```tsx
  import statements
  
  interfaces/types
  
  constants
  
  main component
  
  helper components (if any)
  
  styles
  ```

### Styling
- **StyleSheet API**: Use React Native's `StyleSheet.create()` for component styles
- **No inline styles**: Avoid inline style objects except for dynamic values
- **Design system**:
  - Headers: 28px, Bold
  - Titles: 20px, Bold  
  - Body: 15px, Medium
  - Small: 13px, Regular
- **Color scheme**:
  - Primary: `#0066CC` (blue)
  - Success: `#4CAF50` (green)
  - Warning: `#FF9800` (orange)
  - Error: `#FF6B6B` (red)
  - Background: `#F5F7FA`
- **Meal type colors**:
  - Breakfast: Orange/Yellow
  - Lunch: Green
  - Dinner: Blue/Purple
  - Snacks: Pink/Red

### Component Patterns
- **SafeAreaView**: Always wrap screens with `SafeAreaView` from `react-native-safe-area-context`
- **KeyboardAvoidingView**: Use for screens with input fields
- **ScrollView**: Use for scrollable content with `refreshControl` for pull-to-refresh
- **Loading states**: Show `AILoader` component during async operations
- **Error handling**: Use toast notifications for user feedback

### State Management
- **Context API**: Use React Context for global state (HealthContext)
- **Local state**: Use `useState` for component-local state
- **AsyncStorage**: Use for data persistence (user profile, meal plans, logs)
- **State updates**: Always immutable updates, use spreading operators

## Medical and AI Guidelines

### Medical Safety
- **Disclaimers**: Always show medical disclaimers when displaying health advice
- **Data validation**: Validate all health-related inputs (age, weight, height, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niksbanna/medidiet-app](https://github.com/niksbanna/medidiet-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
