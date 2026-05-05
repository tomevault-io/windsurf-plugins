---
trigger: always_on
description: Substreamer is a React Native music streaming client for Subsonic-compatible servers (Subsonic, Navidrome, etc.), built with Expo SDK 55, React 19, and TypeScript (strict mode).
---

# Substreamer – Project Overview

Substreamer is a React Native music streaming client for Subsonic-compatible servers (Subsonic, Navidrome, etc.), built with Expo SDK 55, React 19, and TypeScript (strict mode).

## Tech Stack

- **Framework:** Expo ~55 / React Native 0.83 (New Architecture enabled)
- **Routing:** Expo Router (file-based) with Stack + Tab navigators
- **State:** Zustand with SQLite persistence (`expo-sqlite`)
- **API:** `subsonic-api` library for Subsonic REST protocol
- **Audio:** `react-native-track-player` (RNTP, local fork in `modules/`) for streaming and background playback
- **Lists:** `@shopify/flash-list` v2 (FlashList) for all performant lists – replaces React Native FlatList
- **Image caching:** Custom disk cache via `expo-file-system`
- **Animations:** `react-native-reanimated` (v4) for all animations – see `ux-quality` rule for details and exceptions
- **i18n:** `react-i18next` v17 + `i18next` v26 with `@formatjs/intl-pluralrules` polyfill for Hermes
- **Styling:** `StyleSheet.create` + inline theme colors (no CSS-in-JS libraries)
- **Path alias:** `@/*` maps to `./src/*`

## Directory Structure

```
src/
  app/            # Expo Router routes (thin wrappers that import from screens/)
    _layout.tsx   # Root Stack layout, auth guard, splash screen
    (tabs)/       # Bottom tab navigator group
    album/[id]    # Dynamic routes for entities
    artist/[id]
    playlist/[id]
  screens/        # Screen components with business logic
  components/     # Reusable UI components
  hooks/          # Custom hooks
  services/       # API clients and external integrations
  store/          # Zustand stores
  i18n/           # react-i18next singleton, locale JSON files, language list
  constants/      # Theme definitions
  utils/          # Formatting, color, string, and timing helpers
  assets/         # App icons, splash images
modules/          # Local Expo native modules (see native-modules rule)
scripts/          # Build helper scripts
fastlane/         # Store listing metadata (descriptions, screenshots, release notes)
  metadata/       # Plain-text metadata files for iOS and Android stores
```

## Key Architectural Patterns

1. **Route/Screen separation:** Route files in `app/` are thin wrappers; business logic lives in `screens/`. See `routing-and-navigation` rule.
2. **Zustand stores** manage all app state. See `zustand-stores` rule.
3. **Services** are plain modules exporting async functions (no classes). See `services-and-api` rule.
4. **CachedImage** is the standard component for all cover art – never use raw `<Image>` for Subsonic artwork.
5. **`useTheme()`** provides `{ theme, colors }` – all components consume colors from this hook rather than importing theme constants directly.
6. **Shared utilities** live in `src/utils/` – common helpers (alphabet indexing via `getFirstLetter`, minimum-delay promises via `minDelay`) are extracted here rather than duplicated.
7. **All user-facing strings** are translated via `react-i18next`. See `Internationalization` section below.

## Internationalization (i18n)

All user-facing strings use `react-i18next`. English is the source language; translations are stored as flat JSON in `src/i18n/locales/`.

### Setup

- **Runtime:** `i18next` v26 + `react-i18next` v17 + `i18next-resources-to-backend` for lazy loading
- **Hermes polyfill:** `@formatjs/intl-pluralrules/polyfill-force` — imported first in `src/i18n/i18n.ts` (must precede i18next init)
- **Locale persistence:** `localeStore` (Zustand + SQLite) — `null` = follow device locale
- **Test setup:** `src/test-utils/i18nSetup.ts` initializes i18next with English resources; included in Jest `setupFiles`

### Usage Patterns

**In React components** — use the `useTranslation` hook:

```tsx
import { useTranslation } from 'react-i18next';

function MyScreen() {
  const { t } = useTranslation();
  return <Text>{t('recentlyAdded')}</Text>;
}
```

**In services/stores (outside React)** — import `i18next` directly:

```tsx
import i18n from 'i18next';

processingOverlayStore.getState().showSuccess(i18n.t('playlistCreated'));
```

**Module-level constant arrays** — use `labelKey` instead of `label`:

```tsx
const OPTIONS = [
  { value: 'recent', labelKey: 'recentlyAdded' },
];
// At render: <Text>{t(opt.labelKey)}</Text>
```

**Interpolation:** `t('greeting', { name: 'Miles' })` → key: `"greeting": "Hello {{name}}"`

**Plurals:** Key-suffix convention with `_one`/`_other`:

```json
{ "songCount_one": "{{count}} song", "songCount_other": "{{count}} songs" }
```
```tsx
t('songCount', { count: 5 }) // "5 songs"
```

### Key Naming Rules

- Flat camelCase: `recentlyAdded`, not `home.recentlyAdded`
- Single `translation` namespace (no namespace prefix in `t()` calls)
- Reuse existing keys for shared strings (`cancel`, `delete`, `save`, `albums`, etc.)
- Check `src/i18n/locales/en.json` before creating new keys

### What NOT to Translate

- Remote API data (album titles, artist names, track titles)
- App name "Substreamer"
- Technical identifiers, log messages, file paths
- Numeric format strings from formatters (`1h30m`, `1.2 GB`)

### Crowdin Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghenry22/substreamer](https://github.com/ghenry22/substreamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
