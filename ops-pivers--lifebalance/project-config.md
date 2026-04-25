---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LifeBalance is a React-based household management application combining finance tracking, habit building, and gamification. Built with Vite, TypeScript, and Tailwind CSS, running on port 3000.

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Environment Setup

Create a `.env.local` file in the project root (copy from `.env.local.example`):

```bash
# Firebase Configuration
VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FIREBASE_MEASUREMENT_ID=your_measurement_id

# Gemini API (for AI features)
VITE_GEMINI_API_KEY=your_gemini_api_key

# Firebase Cloud Messaging (for push notifications)
VITE_FIREBASE_VAPID_KEY=your_vapid_key_here
```

**Required for:**
- Firebase Authentication (Google Sign-In)
- Firestore database persistence and real-time sync
- AI features (Gemini API): receipt scanning, meal suggestions, grocery receipt parsing
- Push notifications (FCM): habit reminders, budget alerts, streak warnings, bill reminders

**Note:** `.env.local` is git-ignored to protect your credentials.

## Architecture

### State Management

The entire application state is managed through a single **React Context**: `FirebaseHouseholdContext` ([contexts/FirebaseHouseholdContext.tsx](contexts/FirebaseHouseholdContext.tsx)).

This context provides:
- **Finance**: Accounts, budget buckets, transactions, calendar items, pay periods
- **Gamification**: Habits, points (daily/weekly/total), challenges, rewards
- **Meals**: Meal recipes, weekly meal planning, shopping lists
- **Safe-to-Spend Calculation**: Real-time financial health metric

All data is persisted in **Firestore** with real-time synchronization across devices using Firebase's `onSnapshot` listeners.

### Safe-to-Spend Logic

The core financial metric (`safeToSpend`) is calculated as:
```
Checking Balance - Unpaid Bills (this month) - Remaining Budget Bucket Limits - Pending Transactions
```

**Critical implementation details:**
- Only checking accounts count as available funds (not savings or credit)
- Bills covered by buckets are excluded to avoid double-counting
- Pending transactions reduce both checking balance and bucket liabilities
- Located in [contexts/HouseholdContext.tsx:258-308](contexts/HouseholdContext.tsx#L258-L308)

### Habit Tracking System

Habits support two scoring modes:

1. **Threshold**: Points awarded only when `targetCount` is reached (e.g., "Read 30 mins" = 1 completion)
2. **Incremental**: Points on every action (e.g., "Late night snack" = -10 pts each time)

**Streak Multipliers:**
- 3-6 days: 1.5x points
- 7+ days: 2.0x points

**Note:** Weather-sensitive bonuses are temporarily disabled. See [WEATHER_IMPLEMENTATION.md](WEATHER_IMPLEMENTATION.md) for future implementation plan.

Habits auto-reset based on their `period` (daily/weekly).

### Routing

Uses **HashRouter** (not BrowserRouter) to support deployment without server-side routing configuration. Routes defined in [App.tsx:21-26](App.tsx#L21-L26).

### External Services

**Gemini API** ([services/geminiService.ts](services/geminiService.ts)):
- **Receipt Scanning**: `analyzeReceipt()` - OCR for expense receipts (model: `gemini-3-flash-preview`)
  - Returns: merchant, amount, category, date
- **Bank Statement Parsing**: `parseBankStatement()` - Extracts transaction lists from screenshots
  - Returns: array of transactions with dates, descriptions, amounts
- **Meal Suggestions**: `suggestMeal()` - AI-powered meal planning based on budget and time constraints
  - Returns: meal name, description, ingredients, tags, reasoning
- **Grocery Receipt Parsing**: `parseGroceryReceipt()` - Extracts grocery items from receipt photos
  - Returns: array of items with name, category, quantity

### Styling

**Tailwind CSS** via CDN (configured in [index.html](index.html)):
- Custom theme colors: `brand-*`, `money-*`, `habit-*`
- Custom fonts: Inter (sans), JetBrains Mono (mono)
- Mobile-first with safe-area-inset support
- No separate Tailwind config file; configuration embedded in HTML `<script>` tag

### Component Organization

```
components/
  ├── auth/         # Authentication components (ProtectedRoute, HouseholdInviteCard)
  ├── budget/       # Budget-specific UI components
  ├── habits/       # Habit tracking UI components
  ├── layout/       # TopToolbar, BottomNav
  ├── meals/        # Meal planning components (MealPlanTab, ShoppingListTab)
  └── modals/       # Modal dialogs for forms

pages/              # Route-level page components
  ├── Dashboard.tsx      # Main overview with AI insights
  ├── Budget.tsx         # Finance management
  ├── Habits.tsx         # Habit tracker
  ├── MealsPage.tsx      # Meal planning and shopping
  ├── Settings.tsx       # App settings and preferences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OPS-PIvers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
