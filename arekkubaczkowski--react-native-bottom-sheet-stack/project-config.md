---
trigger: always_on
description: Provides current sheet ID to children. Used by `useBottomSheetContext`.
---

# CLAUDE.md - LLM Guide for react-native-bottom-sheet-stack

## CRITICAL: React Compiler - NO MANUAL MEMOIZATION

**This project uses React Compiler (`babel-plugin-react-compiler` v1.0.0) with React 19.**

### DO NOT USE:
- `React.memo()`
- `useMemo()`
- `useCallback()`
- `memo()` HOC
- Any manual memoization patterns

### WHY:
React Compiler automatically handles all memoization at build time. Manual memoization is:
1. **Redundant** - Compiler does it better
2. **Harmful** - Can conflict with compiler optimizations
3. **Unnecessary** - Compiler tracks dependencies automatically

### Babel Configuration (babel.config.js):
```javascript
plugins: [
  ['babel-plugin-react-compiler', {
    target: '19',
    panicThreshold: 'all_errors',  // Strict mode
  }],
]
```

### When Compiler Cannot Optimize:
Use the `'use no memo'` directive at the top of the file (see `BottomSheetPortal.tsx` for example). This is RARE and only needed when:
- Dynamic ref cloning breaks compiler analysis
- External library integration requires it

---

## Project Overview

A library-agnostic stack manager for bottom sheets and modals in React Native. Provides:
- **Adapter architecture**: Pluggable adapters for any bottom sheet/modal library
- **Navigation modes**: push, switch, replace
- **iOS-style scale animations**: Background content scales when sheets open
- **Context preservation**: Via portals (`react-native-teleport`)
- **Persistent sheets**: Pre-mounted sheets that maintain state across open/close cycles
- **Type-safe APIs**: TypeScript with augmentable type registry

### Tech Stack (core)
| Category | Package | Version |
|----------|---------|---------|
| React | react | 19.1.0 |
| React Native | react-native | 0.81.5 |
| Animation | react-native-reanimated | ^4.2.1 |
| State | zustand | ^5.0.3 |
| Portals | react-native-teleport | ^0.5.6 |

### Shipped Adapters (separate subpath exports)
| Adapter | Import subpath | Wraps |
|---------|---------------|-------|
| `GorhomSheetAdapter` | `react-native-bottom-sheet-stack/gorhom` | `@gorhom/bottom-sheet` |
| `CustomModalAdapter` | `react-native-bottom-sheet-stack` (main) | Custom animated modal (zero deps) |
| `ReactNativeModalAdapter` | `react-native-bottom-sheet-stack/react-native-modal` | `react-native-modal` |
| `ActionsSheetAdapter` | `react-native-bottom-sheet-stack/actions-sheet` | `react-native-actions-sheet` |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    BottomSheetManagerProvider                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    PortalProvider                         │   │
│  │  ┌────────────────────────────────────────────────────┐  │   │
│  │  │          BottomSheetManagerContext                  │  │   │
│  │  │  (groupId, scaleConfig)                            │  │   │
│  │  └────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┴────────────────────┐
         ▼                                         ▼
┌─────────────────────┐                 ┌─────────────────────┐
│  BottomSheetScaleView │                 │   BottomSheetHost   │
│  (wraps app content)  │                 │   (renders sheets)  │
└─────────────────────┘                 └─────────────────────┘
                                                   │
                    ┌──────────────────────────────┴──────────────────────────────┐
                    ▼                              ▼                              ▼
          ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
          │    QueueItem    │          │    QueueItem    │          │    QueueItem    │
          │  (sheet slot)   │          │  (sheet slot)   │          │  (sheet slot)   │
          │   zIndex: 0,1   │          │   zIndex: 2,3   │          │   zIndex: 4,5   │
          └─────────────────┘          └─────────────────┘          └─────────────────┘
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
┌─────────────────┐   ┌─────────────────┐
│ PortalHost      │   │ Inline Content  │
│ (portal mode)   │   │ (dynamic mode)  │
└─────────────────┘   └─────────────────┘
```

---

## Source File Guide (`src/`)

### Core State Management

#### `bottomSheet.store.ts` - Central Zustand Store
**Purpose**: Single source of truth for all sheet state and stack ordering.

**State Structure**:
```typescript
interface BottomSheetStoreState {
  sheetsById: Record<string, BottomSheetState>;  // All sheets by ID
  stackOrder: string[];                           // Visible sheet IDs in order
}

interface BottomSheetState {
  groupId: string;              // Manager group ID
  id: string;                   // Unique sheet ID
  content?: ReactNode;          // For inline mode only
  status: BottomSheetStatus;    // 'opening' | 'open' | 'closing' | 'hidden'
  scaleBackground?: boolean;    // Enable iOS-style scale
  usePortal?: boolean;          // Portal mode flag
  params?: Record<string, unknown>;  // Type-safe params

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arekkubaczkowski/react-native-bottom-sheet-stack](https://github.com/arekkubaczkowski/react-native-bottom-sheet-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
