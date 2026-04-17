---
trigger: always_on
description: This is a 30-day Detox Fall 2025 wellness tracking app for Russian-speaking users. It helps users track their daily progress through a detox protocol with meal logging, supplement tracking, and progress visualization.
---

# Fall 2025 Detox Tracker - Cursor AI Rules

## Project Overview

This is a 30-day Detox Fall 2025 wellness tracking app for Russian-speaking users. It helps users track their daily progress through a detox protocol with meal logging, supplement tracking, and progress visualization.

## Tech Stack

- **Frontend Framework**: Angular 20.3.6 (standalone components)
- **Styling**: Tailwind CSS 4.x with custom cream/brand color palette
- **UI Components**: @tailwindplus/elements for dropdowns, modals, etc.
- **Backend**: Firebase
  - **Authentication**: Firebase Auth (email/password)
  - **Database**: Firestore (region: nam5 - Iowa)
  - **Image Storage**: Cloudinary (not Firebase Storage)
- **Charts**: Chart.js with ng2-charts
- **Icons**: ng-icons with Heroicons (outline style)
- **Language**: Russian UI (Cyrillic text)

## Project Structure

```
src/app/
├── core/
│   ├── config/          # App configuration (icons.config.ts)
│   ├── guards/          # Auth guards (authGuard, publicGuard)
│   ├── models/          # TypeScript interfaces (UserProfile, DailyLog, Product)
│   ├── services/        # Business logic services
│   └── data/            # Static data (products catalog, nutrition data)
├── features/            # Feature modules (auth, home, daily-log, profile, etc.)
├── layout/              # Shell components (sidebar, top-bar, dashboard-shell)
└── app.routes.ts        # Route configuration
```

## Architecture Patterns

### Authentication Flow

1. **Firebase Authentication** handles login/signup
2. **UserService** manages user profiles in Firestore
3. Auth state changes trigger profile load/clear automatically
4. Guards protect routes: `authGuard` for authenticated routes, `publicGuard` for public routes

### User Profile

- **Model**: `UserProfile` interface with firstName, lastName, country, email, startDate, products
- **Storage**: Firestore collection `userProfiles` (document ID = Firebase Auth UID)
- **Cache**: localStorage cache for instant page loads (key: `detox_user_profile`)
- **Service**: `UserService` handles all profile CRUD operations
- **Cache-First Strategy**: 
  1. Load from localStorage immediately (instant display)
  2. Fetch from Firestore in background to verify/update cache
  3. All profile updates save to both Firestore and localStorage
  4. Logout clears both Firestore cache and localStorage
- **Auto-sync**: Profile loads on login with cache-first strategy, clears on logout via callback pattern
- **Products Field**: Array of product IDs user selected in onboarding (e.g., ['vmg-plus', 'eo-mega'])
- **Cross-device sync**: When user logs in on different device, Firestore provides latest data and updates local cache

### Data Services

- Use Angular signals for reactive state (`signal<T>()`)
- Async operations return Promises
- Services are `@Injectable({ providedIn: 'root' })`
- Firestore integration uses `@angular/fire/firestore`
- **StorageService**: Wrapper for localStorage operations (setItem, getItem, removeItem) with error handling
- **Caching Strategy**: User profiles cached in localStorage for instant page loads, with background sync from Firestore

### Key Services

**UserService** (`core/services/user.service.ts`):
- **userProfile signal**: Reactive state for current user's profile
- **loadUserProfileWithCache(userId)**: Private method using cache-first strategy
  - Step 1: Load from localStorage (instant)
  - Step 2: Fetch from Firestore in background (sync)
- **setUserProfile(profile, userId?)**: Save profile to Firestore + localStorage
- **clearUserProfile()**: Clear profile from signal + localStorage (logout)
- **deleteUserProfile()**: Delete from Firestore + localStorage (account deletion)
- **getStartDate()**: Returns user's detox start date
- **Cache Key**: `detox_user_profile`

**StorageService** (`core/services/storage.service.ts`):
- **setItem<T>(key, value)**: Save to localStorage with JSON serialization
- **getItem<T>(key)**: Retrieve from localStorage with JSON parsing
- **removeItem(key)**: Remove from localStorage
- Error handling for all operations (catches localStorage exceptions)

**ProtocolService** (`core/services/protocol.service.ts`):
- Uses computed signals to access user's startDate and products from UserService
- **getDayNumber(date)**: Returns current day 1-30 in program
- **getWeekNumber(date)**: Returns current week 1-4
- **getCurrentPhase(date)**: Returns phase metadata with Russian descriptions
- **getSupplementsByTiming(date)**: Returns supplements grouped by morning/lunch/dinner (filtered by user's products)
- **isCurrentWeekLocked(date)**: Checks if schedule is available

**DailyLogService** (`core/services/daily-log.service.ts`):
- Local storage management for daily logs
- Save and retrieve daily tracking data (habits, symptoms, supplements, notes)

**ImageService** (`core/services/image.service.ts`):
- Cloudinary image URL generation with transformations
- Product images, profile avatars (future), general assets
- Fallback to local assets if Cloudinary not configured

**AuthService** (`core/services/auth.service.ts`):
- Firebase Authentication wrapper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mashablair) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
