---
trigger: always_on
description: Handles saving frames and metadata after capture triggers.
---

# CLAUDE.md — SnoutCloud (Dog Nose Biometric App)

> Read this file at the start of every session.

---

## Project Overview

**SnoutCloud** is a React Native / Expo Go mobile app that serves as the UI layer for a dog nose biometric identification system. It was converted from a Next.js prototype and shares the same visual design.

**Companion ML repo:** `D:/biometrics_v2/` (NoseEncoder SupCon pipeline)

### Two screens:
1. **Home** — Landing page ("Know Your Dog, Always.") with CTA to scan
2. **Results** — Match found display with dog card, confidence bar, owner info

Camera integration (Phase 2) will add a scan flow between Home → Results.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | React Native via Expo SDK 54 |
| Navigation | React Navigation 7 (`@react-navigation/native-stack`) |
| Storage | `@react-native-async-storage/async-storage` |
| Icons | `@expo/vector-icons` (wraps react-native-vector-icons) |
| Fonts | `@expo-google-fonts/cormorant-garamond` |
| Gradients | `expo-linear-gradient` |
| Animations | React Native `Animated` API |
| TypeScript | Yes |

---

## File Map

```
App.tsx                          Root — font loading, splash, navigation mount
src/
  design/
    tokens.ts                    Design system: colors, typography, spacing, shadows, radii
  navigation/
    index.tsx                    Stack navigator + RootStackParamList type
  screens/
    HomeScreen.tsx               Landing page
    ResultsScreen.tsx            Match results page
  components/
    ProgressBar.tsx              Animated confidence progress bar
assets/
  dog.png                        Dog illustration (484×605 PNG, same as Next.js)
  icon.png                       App icon
  splash-icon.png                Splash screen icon
```

---

## Design System (`src/design/tokens.ts`)

All visual constants are centralized here. Never hardcode colors or sizes in components.

### Color Palette

```typescript
Colors.primaryBrown    = '#905d3c'   // Brand — CTA buttons, logo, headings
Colors.secondaryBrown  = '#c07a52'   // Progress bar gradient end
Colors.darkGreen       = '#2c3520'   // Body text, icons
Colors.mediumGreen     = '#44532e'   // Secondary text, badges
Colors.backgroundWarm  = '#f7f3f0'   // Main background
Colors.backgroundCard  = '#f0e8e9'   // Card / section background
Colors.accentPink      = '#e5d6d7'   // Borders, banner gradient
Colors.white           = '#ffffff'
```

Semi-transparent variants follow the pattern `Colors.primaryBrown70`, `Colors.mediumGreen50`, etc.

### Typography

Font family: **Cormorant Garamond** (serif — loaded via Expo Google Fonts)

```typescript
FontFamily.light    = 'CormorantGaramond_300Light'
FontFamily.regular  = 'CormorantGaramond_400Regular'
FontFamily.medium   = 'CormorantGaramond_500Medium'
FontFamily.semiBold = 'CormorantGaramond_600SemiBold'
FontFamily.bold     = 'CormorantGaramond_700Bold'
```

### Shadows

Pre-built shadow objects — apply with spread operator:
```typescript
style={[styles.myView, Shadows.card]}
```

Available: `logo`, `ctaResting`, `card`, `avatar`, `tabPill`, `actionBtn`

---

## Navigation

Stack with no header (custom headers in each screen):

```
Home → Results (navigate)
Results → Home (goBack)
```

Type-safe navigation with `RootStackParamList`:
```typescript
export type RootStackParamList = {
  Home: undefined;
  Results: undefined;
};
```

Use `useNavigation<HomeNavProp>()` in screens, where `HomeNavProp = NativeStackNavigationProp<RootStackParamList, 'Home'>`.

---

## Animations (React Native Animated API)

| Original CSS | RN Equivalent |
|---|---|
| `animate-scale-in` | `useScaleIn()` hook — opacity + scale + translateY, 550ms bezier |
| `animate-fade-in-up` | `useFadeInUp(delay)` hook — opacity + translateY |
| `animate-float` | `useFloat()` — looping translateY ±10px over 3s |
| `animate-pulse-glow` | `usePulseGlow()` — looping opacity + scale over 2.5s |
| Progress bar fill | `Animated.timing` on width interpolation, 1000ms |

All animations use `useNativeDriver: true` where possible (opacity/transform). The progress bar `width` interpolation requires `useNativeDriver: false`.

---

## AsyncStorage Usage

For future use — storing user preferences, scan history, auth tokens:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Write
await AsyncStorage.setItem('key', JSON.stringify(value));

// Read
const raw = await AsyncStorage.getItem('key');
const value = raw ? JSON.parse(raw) : null;

// Remove
await AsyncStorage.removeItem('key');
```

Currently no AsyncStorage calls exist — will be added in camera/auth phases.

---

## Run Commands

```bash
# Start for Expo Go (scan QR code on device)
npx expo start

# Platform-specific (still uses Expo Go)
npx expo start --ios
npx expo start --android

# Clear cache if needed
npx expo start --clear
```

**Testing on device:** Install Expo Go from the App Store / Google Play, then scan the QR code from `npx expo start`.

---

## Phase Roadmap

- [x] Phase 0 — Screen UI (Landing + Results, exact Next.js parity)
- [ ] Phase 1 — Camera layer (expo-camera, nose scan flow)
- [ ] Phase 2 — API integration (call NoseEncoder inference endpoint)
- [ ] Phase 3 — Auth (login/register, AsyncStorage tokens)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sohambhaumik91/SnoutCloud](https://github.com/sohambhaumik91/SnoutCloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
