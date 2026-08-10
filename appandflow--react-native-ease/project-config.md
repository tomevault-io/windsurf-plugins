---
trigger: always_on
description: `react-native-ease` is a React Native library that provides declarative, native-powered animations via a single `EaseView` component. It uses Core Animation (iOS) and ObjectAnimator/SpringAnimation (Android) — no JS animation loop, no worklets, no C++ runtime.
---

# Agent Instructions for react-native-ease

## Overview

`react-native-ease` is a React Native library that provides declarative, native-powered animations via a single `EaseView` component. It uses Core Animation (iOS) and ObjectAnimator/SpringAnimation (Android) — no JS animation loop, no worklets, no C++ runtime.

**Fabric (new architecture) only.** Does not support the old architecture.

## Project Structure

```
src/                          # TypeScript source (library)
  EaseView.tsx                # React component — flattens props to native
  EaseViewNativeComponent.ts  # Codegen spec — defines native props/events
  types.ts                    # Public TypeScript types
  index.tsx                   # Public exports
  __tests__/                  # Jest tests

ios/
  EaseView.h                  # Native view header (ObjC++)
  EaseView.mm                 # Native view implementation (Core Animation)

android/src/main/java/com/ease/
  EaseView.kt                 # Native view (ObjectAnimator/SpringAnimation)
  EaseViewManager.kt          # ViewManager with @ReactProp setters
  EasePackage.kt              # Package registration

example/                      # Demo app (separate workspace)
  src/App.tsx                 # Main demo screen with animation examples
  src/ComparisonScreen.tsx    # Comparison with Reanimated
  src/components/             # Shared demo components (Section, Button, TabBar)
```

## Architecture

The JS component (`EaseView.tsx`) takes structured props (`animate`, `transition`, etc.) and flattens them into individual native props defined in the codegen spec (`EaseViewNativeComponent.ts`). When those flat props change on the native side, the native view diffs previous vs new values and creates platform-native animations.

**Prop flattening example:**
```
animate={{ opacity: 0.5, scale: 1.2 }}  →  animateOpacity=0.5, animateScale=1.2
transition={{ type: 'spring', damping: 10 }}  →  transitionType="spring", transitionDamping=10
```

**Key design pattern:** All animation logic lives on the native side. The JS layer is purely a prop resolver — no animation state, no timers, no refs.

## Adding a New Animatable Property

1. Add to `AnimateProps` in `src/types.ts`
2. Add `animate<Prop>` and `initialAnimate<Prop>` codegen props in `src/EaseViewNativeComponent.ts`
3. Add default to `IDENTITY` in `src/EaseView.tsx` and pass the flat props to `NativeEaseView`
4. **iOS:** Handle the new property in `updateProps:` — diff old/new, read presentation value, create animation
5. **Android:** Add `pending<Prop>` field, `@ReactProp` setter in `EaseViewManager.kt`, and handle in `applyAnimateValues()`
6. **Recycle:** Reset the new property to its identity value in `prepareForRecycle` (iOS) and `cleanup()` (Android). Fabric recycles views — any property not reset will leak stale values to the next user of the view.
7. Add tests and update README
8. Add an example/demo in the example app (`example/src/App.tsx` or a new screen)

**Important:** When adding or changing props/features, also update:
- `README.md` (props table + usage section)
- `docs/docs/usage.mdx` (usage guide)
- `docs/docs/api-reference.mdx` (API reference table)
- `skills/react-native-ease-refactor/SKILL.md` (supported properties list, transition category keys, decision tree)

## Fixing a Bug

When fixing a reported bug, add a reproducer to the example app so we can manually verify the fix and catch regressions later.

1. Create a new layout under `example/app/issues/<issue-number>/` — use whatever structure the bug needs (tabs, modals, nested stacks). Self-contained so it can be opened directly from the home screen.
2. Register it in `example/src/demos/index.ts` under section `'Issues'` with a `route` field pointing at the new layout (e.g. `route: '/issues/42'`). No `component` is needed — the route owns its own rendering.
3. Include enough context in the screen itself: title, GitHub issue link, repro steps, and visual cues so it's obvious whether the bug is present (e.g. a `mountId` to distinguish "remount" from "still mounted but broken").
4. The reproducer should fail visibly on the broken code path and pass once the fix is in. Don't skip this even when the bug feels obvious — the regression test pays for itself the first time someone touches related code.

## Development Commands

```sh
yarn                          # Install dependencies
yarn test                     # Jest tests (JS layer)
yarn lint                     # ESLint + TypeScript check
yarn format:check             # Prettier + clang-format check
yarn format:write             # Auto-fix formatting
yarn prepare                  # Build JS module (bob build)
yarn example start            # Start Metro for example app
yarn example ios              # Build and run example on iOS
yarn example android          # Build and run example on Android
```

## Pre-commit Checklist

Before committing, always run:

```sh
yarn format:write && yarn lint && yarn test
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appandflow/react-native-ease](https://github.com/appandflow/react-native-ease) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
