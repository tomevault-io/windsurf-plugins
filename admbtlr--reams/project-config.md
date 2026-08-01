---
trigger: always_on
description: ZENITE=true (Rozenite Metro plugin)
---

ZENITE=true (Rozenite Metro plugin)
yarn clear          # same, but clears Metro cache

# Platform shortcuts
yarn ios
yarn android

# Web
yarn deploy:web     # Vercel build + deploy

# Clean slate
yarn nuke           # rm node_modules, reinstall, reinstall pods
```

### Gulp asset server

Article bodies are rendered in embedded WebViews. In dev mode the injected CSS/JS is served by a local Gulp server (`gulpfile.js`). `yarn start` launches it automatically via `concurrently`. Without it, in-app article rendering will be broken.

### Environment

Copy `.env.example` to `.env`. Key variables:
- `EXPO_PUBLIC_API_URL` — URL of the reams-server (Next.js backend, hosted on Vercel)
- Supabase project URL + anon key
- Fastmail JMAP token + mailbox ID (for newsletters)

### External services required

1. **reams-server** — companion Next.js backend (separate repo), runs on Vercel
2. **Supabase** — database; schema in `supabase/schema.sql`
3. **Fastmail** (or any JMAP-compatible account) — newsletter ingestion

---

## Tech Stack

| Area | Technology |
|------|------------|
| Framework | React Native 0.79 + Expo 53 |
| Language | TypeScript (`strict: true`, `allowJs: true`) — mix of `.ts`/`.tsx` and legacy `.js` |
| Navigation | `@react-navigation` v7 (Stack + NativeStack + Drawer) |
| State — global | Redux via RTK `configureStore` + `redux-persist` v5 |
| State — async | `redux-saga` (being gradually replaced by RTK thunks + listener middleware) |
| State — carousel | Zustand (`components/ItemCarousel/bufferedItemsStore.ts`) |
| Database | `expo-sqlite` with FTS5 (native) / IndexedDB via `idb` (web) |
| Styling | `StyleSheet.create` + named HSL colour system (`utils/colors.js`) |
| Article renderer | Embedded `react-native-webview` with injected JS + SCSS (served by Gulp) |
| Build | EAS (dev / preview / production profiles) + Fastlane for iOS |
| Tests | Jest + `jest-expo` + `@testing-library/react-native`; Detox for E2E |
| Analytics | Mixpanel + Sentry |
| Subscriptions | RevenueCat (`react-native-purchases`) |

---

## Project Structure

```
reams/
├── components/          # React components (screens, UI, article viewer)
│   ├── Item/            # Article reader (WebView wrapper + scroll handling)
│   ├── ItemCarousel/    # Swipeable article carousel + Zustand buffer
│   ├── ItemTitle/       # Per-article typographic title renderer
│   ├── onboarding/      # Onboarding flow
│   ├── web/             # Web-only navigation and layout
│   └── App.tsx          # Root navigator (Stack + Drawer)
├── containers/          # Legacy Redux connect() HOC wrappers
├── store/               # Redux store
│   ├── annotations/     # RTK slice ✓
│   ├── categories/      # RTK slice ✓ (+ dead legacy reducer — see below)
│   ├── config/          # Hand-written reducer + action types
│   ├── feeds/           # Hand-written reducer + action types
│   ├── hostColors/      # RTK slice ✓
│   ├── items/           # Hand-written reducers (unread / saved / meta)
│   ├── newsletters/     # RTK slice ✓
│   ├── ui/              # Hand-written reducer
│   ├── user/            # Hand-written reducer
│   ├── index.ts         # configureStore, persistor, sagaMiddleware setup
│   ├── listenerMiddleware.ts  # RTK listener — bridges bootstrap to new thunks
│   ├── migrations.ts    # 28 redux-persist migrations (v0–v27)
│   └── reducers.ts      # combineReducers + reduceReducers cross-slice handler
├── sagas/               # redux-saga side-effect workers
├── backends/            # API adapters (Feedbin, Reams/Supabase, Readwise, Fastmail)
├── storage/             # Platform storage facade (SQLite native / IndexedDB web)
├── hooks/               # useColor, useHeaderStyle
├── utils/               # Colours, dimensions, item style generation, etc.
├── webview/             # JS + SCSS injected into the article WebView
├── supabase/            # Supabase schema + edge functions
├── e2e/                 # Detox E2E tests
└── __tests__/           # Unit tests (backends, components, reducers, sagas, utils)
```

---

## State Management

This is the most complex part of the codebase and is **actively being migrated** from a legacy pattern to RTK. Understanding both patterns is essential.

### Store shape

```
configureStore (RTK)
  ├── downloadsListenerMiddleware  (prepended — RTK createListenerMiddleware)
  ├── sagaMiddleware               (prepended — redux-saga)
  └── RTK default middleware       (serializableCheck: false, immutableCheck: false)

persistReducer (redux-persist v5, current version: 27)
  └── reduceReducers(combineReducers(...), crossSliceReducer)
        ├── itemsUnread       ← hand-written switch reducer
        ├── itemsSaved        ← hand-written switch reducer
        ├── itemsMeta         ← hand-written switch reducer
        ├── feeds             ← hand-written switch reducer
        ├── feedsLocal        ← hand-written switch reducer
        ├── ui                ← hand-written switch reducer
        ├── remoteActionQueue ← hand-written switch reducer
        ├── config            ← hand-written switch reducer
        ├── user              ← hand-written switch reducer
        ├── categories        ← RTK createSlice ✓
        ├── annotations       ← RTK createSlice ✓
        ├── newsletters       ← RTK createSlice ✓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [admbtlr/reams](https://github.com/admbtlr/reams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
