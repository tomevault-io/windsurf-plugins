---
trigger: always_on
description: @huymobile/react-native-iconify is a React Native icon library with 200,000+ icons, native caching via SDWebImage (iOS) and Glide (Android), and zero-configuration setup. It supports both Old and New Architecture (Bridgeless mode).
---

# Cursor Rules for @huymobile/react-native-iconify

## Project Overview
@huymobile/react-native-iconify is a React Native icon library with 200,000+ icons, native caching via SDWebImage (iOS) and Glide (Android), and zero-configuration setup. It supports both Old and New Architecture (Bridgeless mode).

**Key Stats**: v1.0.4, MIT License, TurboModule integration, production icon bundling

---

## Architecture Principles

### 1. Icon Rendering Pipeline
- **Bundled Icons** (production) → Instant render (0ms)
- **Cache HIT** → No loading spinner
- **Cache MISS** → Show spinner, fetch from API, cache result

**Always check bundled icons FIRST**, then cache, then fetch.

### 2. Native Caching Layer
- **iOS**: SDWebImage (~5.10-5.21 compatible) for memory/disk caching
- **Android**: Glide 4.16.0 for memory/disk caching
- **JavaScript**: TurboModule bridge with retry logic

**Cache is the performance foundation** - never bypass it.

### 3. New Architecture Support (Bridgeless Mode)
**Critical**: Lazy-load `react-native-svg` in IconifyIcon.tsx
```typescript
let SvgXml: any = null;
function getSvgXml() {
  if (!SvgXml) {
    SvgXml = require("react-native-svg").SvgXml;
  }
  return SvgXml;
}
```
Top-level imports cause PlatformConstants initialization errors in Bridgeless mode.

---

## Directory Structure & Responsibilities

### `/src`
**TypeScript source** - Single source of truth

- `index.ts` - Main exports (IconifyIcon, cache APIs)
- `components/IconifyIcon.tsx` - ⭐ CORE: Main component with rendering logic
- `api/` - Iconify API interaction (fetch.ts, loader.ts with retry)
- `cache/` - Cache abstraction (TurboCache, native bridge)
- `specs/` - TurboModule specs (New Architecture)

### `/lib`
**Generated output** (tsc compilation) - Never edit manually
- Git ignored, regenerated on `npm run prepare`

### `/ios`
**Native caching implementation**

- `TurboCacheModule.m` - SDWebImage integration
- `Iconify.podspec` - CocoaPods config + script phase for auto-bundling

**Important**: SDWebImage dependency is relaxed (~5.10) for expo-image compatibility

### `/android`
**Native caching + build integration**

- `TurboCacheModule.kt` - Glide integration
- `build.gradle` - ⚠️ REMOVED `com.facebook.react` plugin (fixes duplicate class errors)

### `/scripts`
**Production workflows**

- `bundle-production.js` - Scans icons → fetches → generates bundled-icons.generated.ts
- `scan-icons.js` - Find all IconifyIcon usage in codebase
- `postinstall.js` - Setup after npm install

### `/apps/example-expo`
**Monorepo testing app**

- `metro.config.js` - ⚠️ CRITICAL: Forces app's React instance (fixes "Invalid hook" errors)
- Expo New Architecture enabled for testing

---

## Critical Fixes & Constraints

### 1. Bridgeless Mode (New Architecture iOS)
**Issue**: PlatformConstants not initialized before react-native-svg import
**Solution**: Lazy-load with `require()` in getSvgXml()
**Rule**: NEVER import SvgXml at top-level

### 2. Multiple React Instances
**Issue**: Invalid hook calls when library uses React 18.2.0, app uses 19.1.0
**Solution**: metro.config.js extraNodeModules + blockList
**Rule**: Keep metro.config.js in sync with all node_modules React instances

### 3. Android Duplicate Class
**Issue**: `com.facebook.react` plugin generates duplicate RNGestureHandlerButtonManagerDelegate
**Solution**: ✅ REMOVED from android/build.gradle
**Rule**: Do NOT re-add `apply plugin: "com.facebook.react"`

### 4. ActivityIndicator Flash
**Issue**: Loading spinner flashes on cache HIT
**Solution**: Check cache BEFORE setting loading state
**Rule**: Use `cacheChecked` state to defer rendering spinner until actual miss

### 5. SDWebImage Version Conflict
**Issue**: expo-image requires ~5.19.1, old code required ~5.21.0
**Solution**: Relaxed to ~5.10 in Ionify.podspec (compatible with 5.10-5.x)
**Rule**: Keep dependency flexible for ecosystem compatibility

---

## Development Guidelines

### When Adding Features
1. ✅ Implement in `/src` (TypeScript)
2. ✅ Update corresponding native module if cache/performance related
3. ✅ Add tests in isolated test files
4. ✅ Update example app if user-facing feature
5. ✅ Document in README and inline JSDoc

### When Modifying Components
- **IconifyIcon.tsx**: Test both Old and New Architecture
- **Cache**: Test on real device (not simulator)
- **Native modules**: Test in example-expo with `newArchEnabled: true`

### Code Standards
- ✅ Lazy-load heavy imports (SVG rendering)
- ✅ Keep component logic synchronized across iOS/Android
- ✅ Error handling with fallback rendering
- ✅ Memory leak prevention in useEffect cleanup
- ✅ Type all external APIs with proper interfaces

**Do NOT**:
- ❌ Add top-level native imports (breaks Bridgeless mode)
- ❌ Directly import React components at module scope if optional
- ❌ Re-add Facebook React plugin to Android
- ❌ Hardcode SDWebImage/Glide versions (breaks compatibility)
- ❌ Skip cache checks for performance

---

## Building & Publishing

### Local Development
```bash
npm run clean && npm run build    # Compile TypeScript
npm run scan                       # Find all icons in codebase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huytdps13400/react-native-iconify](https://github.com/huytdps13400/react-native-iconify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
