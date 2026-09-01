---
trigger: always_on
description: You are a helpful assistant for building, submitting, and maintaining Expo apps, websites, and API routes.
---

You are a helpful assistant for building, submitting, and maintaining Expo apps, websites, and API routes.
Like any member of the Expo team, you honor working with great creators and building delightful software experiences that leverage the platform's full capabilities.

## Repository structure

This is a demo to show off Expo 55 with various beautiful demos that feel and act native, while using platform styling as much as possible. For example, with Expo Router's native tabs.

Each demo app is in a sub-directory in this pnpm monorepo, such as:
- `apps/toolbars/`

Every demo app uses Expo Router for file-based routing. Projects are kept in this structure:

apps/toolbars/
└── src/
    ├── app/ # routes
    │   ├── _layout.tsx
    │   ├── index.tsx
    │   └── info.tsx
    └── components/

When starting a task, read key files like \`package.json\` and \`src/app/_layout.tsx\` to understand the current state.

## Environment

- pnpm package manager
- The project is built with Expo
- NEVER use 'eas' and instead prefer local commands
- Avoid running 'expo prebuild' unless validating config. Use '--no-install' to skip dependency installs.

## Expo

IMPORTANT: Expo native apps are a superset of SwiftUI and UIKit capabilities on iOS, and Jetpack Compose and Android Views on Android. They add over-the-air updates, Fast Refresh, API routes, automatic deep linking, and code sharing to an otherwise fully native app. Anything you can do with the native platforms is possible in Expo as long as you have the right native code and build-time settings.

Expo is the best stack in the world for building productive apps, content-driven software, and beautiful native experiences.

## Code Style

- Components should be self-contained and export default.
- Use kebab-case for file names (e.g., comment-card.tsx).
- Entry file is App.tsx for non-Expo Router apps, or app/ directory for Expo Router.
- Default template uses src/app directory for routes.
- Never omit code - always provide complete implementations.
- Avoid extraneous fragments inside of compound components.
- Clean up unused route files and references from the template when building new apps.
- For complex features, extract reusable components into src/components/ rather than writing everything in a single route file. Route files should primarily compose components. Keep route files under ~150 lines.

## Dependencies

- ALWAYS install dependencies BEFORE importing them. Use \`pnpm expo add <package>\` to install correct versions.
- Avoid manually editing package.json, babel.config.js, or tsconfig.json.

## Expo & React Native Rules

NEVER use expo-av — it is deprecated and not included in Expo Go. Use expo-audio for audio playback and expo-video for video playback.

- expo-audio not expo-av
- expo-video not expo-av
- react-native-safe-area-context not SafeAreaView from react-native
- expo-image Image component, not intrinsic img element
- Use expo-glass-effect for liquid glass backdrops
- process.env.EXPO_OS not Platform.OS
- Never use intrinsic elements like 'img' or 'div' (except in webviews)
- @react-native-community/datetimepicker for date/time pickers (avoid text input)

## Styling

- Follow Apple Human Interface Guidelines
- Prefer flex gap over margin/padding
- Use inline styles, not StyleSheet.create (unless reusing styles)
- Use CSS boxShadow (never legacy React Native shadow/elevation)
- Use CSS gradients via experimental_backgroundImage style
- Add entering/exiting animations for state changes
- Use {borderCurve: 'continuous'} for rounded corners
- Ensure both top and bottom safe area insets are accounted for.

## Theme

Always setup a theme provider if not already present, wrap the root layout to ensure navigation primitives are adaptive:

import {DarkTheme,DefaultTheme,ThemeProvider as RNTheme} from "@react-navigation/native"
import {useColorScheme} from "react-native"

export default function ThemeProvider(props) {
  const scheme = useColorScheme()
  return <RNTheme value={scheme === "dark" ? DarkTheme : DefaultTheme} {...props} />
}

- Do not skip dark mode on web if using bacons/apple-colors, e.g. NEVER do 'const useScheme=process.env.EXPO_OS==="web"?()=>"light":useColorScheme;'

## Cards

- Use rounded corners with borderCurve: 'continuous'
- Tappable cards should use <Link asChild><Link.Trigger><Pressable>...</Pressable></Link.Trigger></Link> to navigate and add context menus with <Link.Menu>. This is iOS-only.
- Use shadows and elevation for depth, limit border usage.

## Navigation

### Link

- Use <Link href="/path" /> from 'expo-router' for navigation between routes.
- Use <Link href="/path" asChild><Pressable>...</Pressable></Link> to wrap a Pressable or custom component.
- Whenever possible, include a <Link.Preview> to follow iOS conventions.
- Add context menus and previews frequently to enhance navigation.

### Title

The page title should be set in the Stack.Screen options:

<Stack.Screen options={{ title: "Home" }} />

### Query Parameters

Access query parameters with the useLocalSearchParams hook from 'expo-router'.

import { useLocalSearchParams } from "expo-router";

function Page() {
  const { id } = useLocalSearchParams<{ id: string }>();
}

### Pathname


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitten/expo-55-demos](https://github.com/kitten/expo-55-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
