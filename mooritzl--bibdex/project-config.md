---
trigger: always_on
description: npx expo start          # Start Metro bundler
---

## Commands

```bash
npx expo start          # Start Metro bundler
npx expo run:ios        # Build and run on iOS
npx expo run:android    # Build and run on Android
npx expo install --fix  # Fix dependency version mismatches
```

## Architecture

- `App.js` — root: GestureHandlerRootView → SafeAreaProvider → ActionSheetProvider → NavigationContainer → 3 native tabs
  - **Home** tab → Stack: `HomeOverviewScreen` (dashboard) → `ProfileScreen` (account) → `LibraryDetailScreen` (favorites tap)
  - **BibDex** tab → Stack: `HomeScreen` (list) → `LibraryDetailScreen` (detail + check-in)
  - **BibMap** tab → `MapScreen` (placeholder)
- `src/store/libraryStore.js` — Zustand store; actions: `setUid`, `subscribeToLibraries`, `subscribeToUserProfile`, `checkIn`, `updateProfile`, `signOut`, `deleteAccount`. Both library subscriptions merge in `MainTabs` via `useEffect`.
- `src/screens/HomeOverviewScreen.js` — discovery hero card (large count + progress + 4 stats), nearby check-in card (5 states), favorites, challenges
- `src/components/CheckInModal.js` — animated reward modal (Reanimated 4.x); gem icon + drain bar; first-discovery vs repeat distinction
- `src/screens/ProfileScreen.js` — avatar with gold ring, trophy stats grid (Playfair numbers), leaderboard row, account/legal settings
- `src/screens/HomeScreen.js` — library list with custom dark filter tabs (All/Discovered/Undiscovered) + discovery progress header (FlatList)
- `src/screens/LibraryDetailScreen.js` — rarity gradient hero with glowing gem icon, XP progress bar, check-in button shows `CheckInModal` then navigates back
- `src/screens/MapScreen.js` — MapView with rarity-colored pins; bottom-right control stack (BlurView group: filter + tracking buttons); content-sized filter sheet (transparent modal + BlurView + spring animation)
- `src/components/LibraryCard.js` — discovered: 2px rarity accent bar + glow shadow; undiscovered: dashed icon + dimmed name
- `src/services/mockData.js` — mock challenges data (libraries now come from Firestore; mockData challenges still used until Firebase challenges are wired)
- `src/theme/colors.js` — light and dark palettes + `RARITY` map + `DISPLAY_FONT` constant
- `src/theme/useTheme.js` — `useTheme()` hook; returns the active palette based on `useColorScheme()`
- `src/components/CardView.js` — generic card wrapper; accepts `style` override; does NOT set `overflow: hidden`
- `src/components/HairlineDivider.js` — `height: StyleSheet.hairlineWidth`
- `src/components/PrimaryButton.js` — props: `backgroundColor`, `icon`, `compact`
- `src/components/InlineProgressBar.js` — track View requires `overflow: hidden`
- `src/components/RarityBadge.js` — dot + uppercase text pill; prop: `rarity` (key into `RARITY` map); `small` variant

State: Zustand (`libraryStore`). Both HomeOverviewScreen and HomeScreen read from the same store.
`totalKP`, `totalCheckIns`, `unlockedCount`, `streak` are **stored aggregates** on `users/{uid}` — mutated with Firestore `increment()`, never recalculated. `checkIn()` is async: optimistic local update first, then `writeBatch` (checkIn event log + libraryProgress upsert + user doc increments); rolls back on error.
Firestore collections: `users/{uid}` (profile + aggregates), `users/{uid}/libraryProgress/{libId}` (per-library state), `users/{uid}/checkIns/{id}` (event log for future stats).
XP/level formula: `level = floor(checkIns/5)+1`, `xp = (checkIns%5)*10`, `xpToNext = 50` (constant).
Streak: computed from `lastCheckInDate` string (`YYYY-MM-DD`) on each check-in write.
Check-in from Home: `expo-location` finds nearest library by Haversine distance; one tap triggers `checkIn(id)` + `CheckInModal`. First unlock = +300 KP; repeat check-in = +50 KP.

## Gotchas

- **Tab navigator**: uses `@bottom-tabs/react-navigation` (native tabs), NOT `@react-navigation/bottom-tabs` — the latter has been removed.
- **Stack navigator**: uses `@react-navigation/native-stack` (`createNativeStackNavigator`), NOT `@react-navigation/stack` — the native stack gives iOS's real swipe-to-go-back with no ScrollView conflicts. Requires `react-native-screens` (already installed).
- **Action Sheet**: `useActionSheet()` from `@expo/react-native-action-sheet` requires `ActionSheetProvider` in `App.js` (already wired). Used in `ProfileScreen` for avatar color picker.
- **CardView overflow**: `CardView` does NOT set `overflow: hidden` — set it on an inner wrapper when needed.
- **SegmentedControl**: installed but no longer used — replaced by custom dark filter tabs in `HomeScreen`.
- **Reanimated peer dep**: `react-native-worklets` (0.5.1) is required by Reanimated 4.x — keep it in `package.json`.
- **CheckInModal**: auto-dismisses after 3400 ms. `isFirst` is derived from `kpGained > 100` (first unlock = 300, repeat = 50).
- **Location snapshot**: Before calling `checkIn(id)`, read the live `isUnlocked` from `libraryList.find(l => l.id === lib.id)` — not from stale state.
- **Mock data**: all libraries are in Munich (real lat/lon). iOS Simulator defaults to Apple HQ — use Features → Location in Simulator to test nearby logic.
- **DISPLAY_FONT**: `Georgia` is used for Playfair Display-style headings/numbers. Import from `src/theme/colors.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moOritzl/BibDex](https://github.com/moOritzl/BibDex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
