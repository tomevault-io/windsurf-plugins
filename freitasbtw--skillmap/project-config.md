---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Development:**
- `npm start` - Start Expo dev server
- `npm run android` - Run on Android emulator/device
- `npm run ios` - Run on iOS (macOS only)
- `npm run web` - Run on web (limited functionality)

**TypeScript:**
- `npx tsc --noEmit` - Type check without building

**Cleaning:**
```bash
rm -rf node_modules package-lock.json
npm install
npx expo start -c  # Clear cache
```

## Environment Setup

Create `.env` file in project root:
```env
OPENAI_API_KEY=sk-proj-...  # Optional - app works in mock mode without it
NODE_ENV=development
```

The app gracefully falls back to mock ChatBot responses if `OPENAI_API_KEY` is not set.

## Architecture Overview

### Persistence Layer Architecture

This app uses a **hybrid persistence strategy** combining three storage mechanisms:

1. **SQLite (expo-sqlite)** - Primary database for all entities
   - Tables: `users`, `roadmaps`, `skills`, `roadmap_skills`, `chat_messages`
   - Managed via singleton `DatabaseService` (`src/services/DatabaseService.ts`)
   - Initialized on app startup in `App.tsx`

2. **SecureStore (expo-secure-store)** - Encrypted authentication tokens only
   - Stores: `AUTH_TOKEN` (user ID for session validation)
   - Never store sensitive data beyond tokens here

3. **AsyncStorage** - Cache layer for performance
   - Stores: `USER_DATA` (non-sensitive user object for quick access)
   - Synced from SQLite after mutations

**Critical Pattern:** All data mutations must update ALL three layers:
```typescript
// Example from AuthService.ts:77-106
await DatabaseService.createUser({...});           // 1. SQLite (source of truth)
await SecureStore.setItemAsync('AUTH_TOKEN', id);     // 2. SecureStore (session)
await AsyncStorage.setItem('USER_DATA', JSON.stringify(user)); // 3. Cache
```

### Authentication Flow

Authentication spans multiple services and contexts:

**Login/Signup:**
1. `AuthService.ts` validates credentials against SQLite
2. SHA-256 password hash comparison via `utils/validation.ts`
3. Session token stored in SecureStore
4. User object cached in AsyncStorage
5. `AuthContext.tsx` updates React state triggering navigation guard

**Session Restoration (App Start):**
1. `useAuth` hook calls `AuthService.verificarSessao()` on mount
2. Token retrieved from SecureStore and validated against SQLite
3. If valid: user restored; if invalid: token cleared and user logged out
4. Navigation guard in `AppNavigator.tsx` routes to login/main based on auth state

**Auth Guard Logic (`AppNavigator.tsx:101-194`):**
```
Auth Flow:
  Loading → Check SecureStore token → Validate in SQLite
    ├─ Invalid/None → Show: OnboardingCadastro → Login → Cadastro
    └─ Valid → Check onboarding status
         ├─ Not seen → Show: OnboardingLogin → MainTabs
         └─ Seen → Show: MainTabs (authenticated)
```

### Navigation Architecture

**Two-tier navigation system:**

1. **Stack Navigator** (AppNavigator.tsx) - Auth-aware routing
   - Conditionally renders auth screens vs. MainTabs based on `user` from AuthContext
   - Handles onboarding flow using AsyncStorage flags keyed by user ID
   - Prevents gestures on authenticated screens (`gestureEnabled: false`)

2. **Bottom Tab Navigator** (MainTabs component) - Authenticated screens only
   - Home, GeradorRoadmap, RoadmapTracker, ChatBot
   - Safe area insets handled for iOS notch/Android navigation

**Onboarding Pattern:**
```typescript
// AppNavigator.tsx:110-142
const onboardingKey = `${STORAGE_KEYS.ONBOARDING}_login_${user.id}`;
const seen = await AsyncStorage.getItem(onboardingKey);
setHasSeenOnboarding(!!seen);
```
Each user tracks onboarding separately. Reset on logout.

### Service Layer Patterns

All services follow these conventions:

**DatabaseService (Singleton):**
- `getInstance()` for single instance across app
- All methods async, return promises
- CRUD methods: `create*`, `get*`, `update*`, `delete*`
- SQL queries use parameterized statements to prevent injection
- Foreign keys with CASCADE deletes maintain referential integrity

**AuthService (Singleton):**
- Does NOT directly access storage - delegates to DatabaseService
- All methods return `AuthResult` or `IUser | null`
- Password operations delegated to `utils/validation.ts`
- Extensive console logging for debugging (remove in production)

**RoadmapService:**
- Handles roadmap generation (currently mock IA logic)
- Coordinates between DatabaseService (roadmaps + skills) and XP system
- Skills awarded on completion: +50 XP per skill, +500 XP per 100% roadmap

### State Management

**No Redux/Zustand** - uses React Context API only:

**AuthContext (`src/contexts/AuthContext.tsx`):**
- Provides: `user`, `isLoading`, `login`, `cadastrar`, `logout`, `atualizarXP`
- Single source of truth for auth state
- Wraps entire app in `App.tsx`

**Local State:**
- Screens use `useState` for UI state (loading, errors, form data)
- Custom hooks (`useAuth`, `useRoadmap`) encapsulate business logic
- No prop drilling - prefer context or component composition

### TypeScript Patterns

**Strict typing enforced:**
- All interfaces in `src/types/models.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freitasbtw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
