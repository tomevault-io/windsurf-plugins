---
trigger: always_on
description: MapLibre React Native provides React Native bindings to MapLibre Native rendering engines for Android & iOS. It wraps native MapView components, camera controls, data sources, and style layers as React components, bridging JavaScript props to native view properties through React Native's new architecture (Fabric + TurboModules).
---

# Agent Guide for MapLibre React Native

## What This Library Does

MapLibre React Native provides React Native bindings to MapLibre Native rendering engines for Android & iOS. It wraps native MapView components, camera controls, data sources, and style layers as React components, bridging JavaScript props to native view properties through React Native's new architecture (Fabric + TurboModules).

**Key Facts**:

- **Current version v11**: Only supports React Native's new architecture (Fabric/TurboModules)
- **Native SDKs**: MapLibre Native iOS v6 ([`package/MapLibreReactNative.podspec`](package/MapLibreReactNative.podspec)), Android v13 ([`package/android/gradle.properties`](package/android/gradle.properties))
- **Runtime**: React Native >=0.80, Node v24 ([`.nvmrc`](.nvmrc)), Yarn 4 (corepack)
- **Languages**: TypeScript (shared), Objective-C (iOS), Kotlin (Android)

## Library Architecture

### Component Structure

- **Map**: Root map container, wraps native MapLibre view via Fabric codegen (`MapViewNativeComponent.ts`)
- **Camera**: Controls viewport (zoom, bearing, pitch, center), uses imperative ref API
- **Sources**: Data providers (GeoJSONSource, VectorSource, RasterSource, ImageSource) - children of Map
- **Layers**: Visual representation (FillLayer, LineLayer, SymbolLayer, etc.) - children of Sources
- **Annotations**: User interaction elements (ViewAnnotation, LayerAnnotation, Marker, Callout)
- **Modules**: Native modules for offline, location, logging, snapshots

### Key Patterns

1. **Fabric Components**: Components using new arch have `*NativeComponent.ts` files with `codegenNativeComponent`
2. **Turbo Modules**: Modules using new arch have `Native*Module.ts` files with `TurboModuleRegistry.getEnforcing`
3. **Accompanied Modules for Components**: Components like `Map` have `MapViewModule` for imperative methods
4. **Style Transformation**: Props → native format via `transformStyle()` in `utils/StyleValue`
5. **Children as Config**: Sources contain Layers, Layers inherit sourceID from parent
6. **Ref-based Imperative API**: Map, Camera, GeoJSONSource expose methods via `useImperativeHandle`

### Codegen System

`scripts/codegen.ts` generates from MapLibre style spec + TSDoc comments:

- **Native style classes**: iOS `.h/.m`, Android `.java` in `components/layers/style/`
- **TypeScript types**: `src/types/MapLibreRNStyles.ts` (layer styles, expressions)
- **Documentation**: `/docs/content/components/` and `/docs/content/modules/`

**NEVER edit generated files** - they have header comments indicating source.

## Code Style & Conventions

### TypeScript

- **Strict mode enabled** (`tsconfig.json`) - no implicit any, unused vars, etc.
- **Export pattern**: Named exports only, barrel exports in `index.ts`
- **Props**: Use `interface` with `Props` suffix (e.g., `MapProps`, `CameraProps`)
- **Types**: Use `type` for unions/mapped types, `interface` for object shapes
- **Ref types**: Pattern: `ComponentRef` (e.g., `MapRef`, `CameraRef`)
- **Native props**: Separate `NativeProps` interface for codegen-compatible types
- **Null safety**: Always check `useRef` values before use, use optional chaining

### Comments

- **Use comments very sparingly** - well-named functions and variables should make code self-explanatory
- **TSDoc for public API**: Use TSDoc (`/** */`) on exported components, functions, and types - this feeds the codegen and documentation pipeline
- **No inline narration**: Do not add comments that simply restate what the code does (e.g., `// loop through items`, `// return result`)
- **Reserve inline comments** for genuinely non-obvious decisions, workarounds, or important caveats that cannot be expressed through naming alone

### Naming Conventions

- **Native modules**: `MLRN` prefix (e.g., `MLRNMapView`, `MLRNCamera`)
- **Files**: PascalCase for components, camelCase for utils/hooks
- **Props**: Descriptive, follows MapLibre terminology (e.g., `bearing`, `pitch`)
- **Events**: `on` prefix (e.g., `onPress`)

### Testing

- Mock native modules in [`package/src/__tests__/__mocks__`](package/src/__tests__/__mocks__)
- Use React Native Testing Library patterns
- Test component prop handling, not native behavior
- E2E tests in Maestro verify native integration

### Path Aliases

**In Jest Tests** ([`package/src/__tests__/`](package/src/__tests__/)):

- `@maplibre/maplibre-react-native`: Public exports
- `@/*`: Internal exports
- Configured in `jest.config.ts`

**In Example Apps** (`examples/shared/`):

- `@/*`: References `examples/shared/src/*`
- Configured in `tsconfig.json`, `metro.shared.js`, and `babel.shared.js`

## When to Edit Which Files

### Adding a New Component

1. Create component in `src/components/` (e.g., `MyComponent.tsx`)
2. If using Fabric: Create `MyComponentNativeComponent.ts` with `codegenNativeComponent`
3. Add exports to `src/index.ts`
4. Add TSDoc comments (triggers codegen for docs)
5. Create native implementations:
   - iOS: ViewManager in `ios/components/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maplibre/maplibre-react-native](https://github.com/maplibre/maplibre-react-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
