---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
npx expo start          # start Metro bundler (press i/a/w for iOS/Android/Web)
npx expo start --ios    # launch iOS simulator directly
npx expo start --android
npm run lint            # expo lint (ESLint)
npm test                # jest --watchAll
cd functions && npm run build   # compile Cloud Functions TypeScript
cd functions && npm run deploy  # deploy Cloud Functions to Firebase
```

There is no separate build step for local dev — Expo handles transpilation at runtime. Cloud builds use EAS (`eas build --profile development|preview|production`).

**EAS build profiles** (`eas.json`):
- `development` — dev client; iOS simulator + Android APK
- `preview` — internal distribution (device install)
- `production` — app store submission with auto-incremented versions

## Environment

Copy `.env.local.example` to `.env.local` and fill in all keys. Client vars are prefixed `EXPO_PUBLIC_` so Expo exposes them to the bundle. Cloud Function vars are set via `firebase functions:config:set` or Firebase environment secrets — never `EXPO_PUBLIC_`.

| Variable | Where | Purpose |
|---|---|---|
| `EXPO_PUBLIC_FIREBASE_API_KEY` | client | Firebase API key |
| `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN` | client | Firebase auth domain |
| `EXPO_PUBLIC_FIREBASE_PROJECT_ID` | client | Firebase project ID |
| `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET` | client | Firebase storage bucket |
| `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | client | Firebase messaging sender ID |
| `EXPO_PUBLIC_FIREBASE_APP_ID` | client | Firebase app ID |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | client | Google Places autocomplete (New API) |
| `EXPO_PUBLIC_REVENUECAT_IOS_KEY` | client | RevenueCat iOS SDK |
| `EXPO_PUBLIC_REVENUECAT_ANDROID_KEY` | client | RevenueCat Android SDK |
| `EXPO_PUBLIC_ALGOLIA_APP_ID` | client | Algolia search app ID |
| `EXPO_PUBLIC_ALGOLIA_SEARCH_KEY` | client | Algolia **Search-Only** key (never Admin) |
| `ALGOLIA_APP_ID` | Cloud Function | Algolia sync (Admin key context) |
| `ALGOLIA_ADMIN_KEY` | Cloud Function | Algolia index write access |
| `AVIATIONSTACK_API_KEY` | Cloud Function | Flight status polling |
| `GEMINI_API_KEY` | Cloud Function | AI trip generation (never expose to client) |

## Architecture

### Routing (Expo Router v6)

File-based routing under `app/`. App bundle ID: `com.supernovatravel.app`. React Native New Architecture is enabled.

```
app/
├── _layout.tsx                    # Root layout — auth listener, RevenueCat init, push token
├── (auth)/
│   ├── _layout.tsx
│   ├── welcome.tsx
│   ├── sign-in.tsx
│   ├── sign-up.tsx
│   └── forgot-password.tsx
├── (tabs)/
│   ├── _layout.tsx                # Tab bar: Feed, Explore, Create, Search, Profile
│   ├── index.tsx                  # Feed
│   ├── explore.tsx
│   ├── create.tsx
│   ├── search.tsx
│   └── profile.tsx
├── (wallet)/
│   ├── _layout.tsx
│   ├── boarding-passes.tsx        # List
│   ├── boarding-pass/[id].tsx     # Detail
│   ├── boarding-pass/add.tsx      # Add form
│   ├── reservations.tsx           # List
│   ├── reservation/[id].tsx       # Detail
│   ├── loyalty.tsx                # List
│   ├── loyalty/[id].tsx           # Detail
│   └── loyalty/add.tsx            # Add form
├── trip/
│   ├── [id].tsx                   # Trip detail (modal)
│   ├── new.tsx                    # Manual trip wizard (modal)
│   ├── ai-generate.tsx            # AI generation form (modal)
│   └── ai-generating.tsx          # Generation loading screen → routes to trip/[id]
├── post/[id].tsx                  # Post detail (modal)
├── user/[uid].tsx                 # Public profile (full-screen push)
├── settings.tsx                   # Theme toggle, account, sign out (modal)
└── paywall.tsx                    # RevenueCat paywall (modal)
```

**Auth routing is centralised in `app/_layout.tsx`** via a single `onAuthStateChanged` listener. On sign-in it calls `configureRevenueCat(uid)`, registers the Expo push token, fetches the user's `tier`, then `router.replace('/(tabs)')`. On sign-out: `router.replace('/(auth)/welcome')`. There is no route guard middleware.

Path alias `@/` maps to the project root (see `tsconfig.json`).

### State Management

Three Zustand stores in `stores/`:
- `useAuthStore` — Firebase `User | null`, `tier: 'free' | 'pro' | 'business'`, initialization flag
- `useUserStore` — cached `UserProfile` (displayName, avatarUrl, bio, location, follower counts)
- `useThemeStore` — `mode: 'dark' | 'light' | 'system'`, persisted via AsyncStorage; `useTheme()` hook resolves `'system'` via `useColorScheme`

TanStack React Query (staleTime 2 min, 2 retries) wraps all Firestore reads. **No `onSnapshot` listeners in hooks** — use `getDocs`/`getDoc` only. Exception: post comments screen uses `onSnapshot` directly in a `useEffect`.

### Firebase

`services/firebase.ts` exports `auth`, `db`, `storage`, `functions` as named singletons (region: `us-central1`). Import these directly — never call `getAuth()` / `getFirestore()` elsewhere.

Firestore collections:
- `users/{uid}` — profile + `tier` + `expoPushTokens[]`; subcollections: `feed/`, `notifications/`, `savedTrips/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markolsen619/supernova-travel](https://github.com/markolsen619/supernova-travel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
