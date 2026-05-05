---
trigger: always_on
description: When you ask Copilot for advice or implementation approach:
---

# Copilot Instructions for BattleDex

## How to Request Help from Copilot

When you ask Copilot for advice or implementation approach:

1. **For design/architecture questions**: Copilot will provide **multiple options/approaches** before writing code
2. **Each option will include**:

- Pros and cons
- When to use it
- Estimated effort

3. **You can then**:

- Choose your preferred approach
- Ask for clarifications
- Request modifications

4. **Only after you confirm**: Copilot will proceed with the code implementation

This ensures better alignment between your vision and the implementation.

## Critical Architecture Rule: Screen Organization

When creating features in this React Native project, follow this pattern:

### Multiple Screens (2+) in a Feature

✅ **Create a separate subdirectory for EACH screen** under `presentation/`

```
src/features/{featureName}/
├── domaine/
├── data/
└── presentation/
    ├── {screenName1}/                    ← Screen 1 directory
    │   ├── {ScreenName1}Screen.tsx
    │   ├── {screenName1}ScreenDI.ts
    │   ├── {screenName1}ScreenRoute.ts
    │   ├── use{ScreenName1}ScreenViewModel.tsx
    │   ├── styles/
    │   │   └── {screenName1}Screen.styles.ts
    │   └── components/
    │       └── styles/
    │
    └── {screenName2}/                    ← Screen 2 directory
        ├── {ScreenName2}Screen.tsx
        ├── {screenName2}ScreenDI.ts
        ├── {screenName2}ScreenRoute.ts
        ├── use{ScreenName2}ScreenViewModel.tsx
        └── styles/
            └── {screenName2}Screen.styles.ts
```

**Example**: `collection` feature has 2 screens:

- `presentation/collection/` - Main collections list
- `presentation/collectionGroup/` - Individual group details

### Single Screen in a Feature

✅ **NO subdirectory** - Keep files flat in `presentation/`

```
src/features/{featureName}/
├── domaine/
├── data/
└── presentation/                         ← NO subdirectory!
    ├── {ScreenName}Screen.tsx
    ├── {screenName}ScreenDI.ts
    ├── {screenName}ScreenRoute.ts
    ├── use{ScreenName}ScreenViewModel.tsx
    ├── styles/
    │   └── {screenName}Screen.styles.ts
    └── components/
        └── styles/
```

**Example**: `card` feature has 1 screen:

- `presentation/CardScreen.tsx` (directly in presentation/)
- `presentation/cardScreenDI.ts`
- `presentation/useCardScreenViewModel.tsx`

## Clean Architecture Layers

```
src/features/{featureName}/
├── domaine/          # Entities & repository interfaces
│   ├── entities/
│   │   └── {EntityName}.ts       ← Business entities
│   └── {RepositoryName}.ts       ← Repository interfaces
├── data/             # Repository implementations
│   └── {RepositoryName}Impl.ts
└── presentation/     # UI components & screens
    └── {screenName}/
        ├── {ScreenName}Screen.tsx
        ├── components/
        └── styles/
```

### Domain Layer (domaine/)

- **entities/**: Define TypeScript interfaces for business entities
  ```typescript
  // domaine/entities/CollectionGroupEntity.ts
  export interface CollectionGroupEntity {
    id: string;
    name: string;
    cardCount: number;
    color: string;
  }
  ```
- **Repository Interfaces**: Define contracts for data access
  ```typescript
  // domaine/SavedCardsRepository.ts
  export interface SavedCardsRepository {
    getSavedCards(): CardEntity[];
    addCard(card: CardEntity): void;
  }
  ```

## Required Files per Screen

Every screen needs:

1. `{ScreenName}Screen.tsx` - Main component (default export, use `observer()` if reactive)
2. `use{ScreenName}ScreenViewModel.tsx` - Business logic hook
3. `{screenName}ScreenDI.ts` - Dependency injection
4. `{screenName}ScreenRoute.ts` - Route configuration
5. `styles/{screenName}Screen.styles.ts` - Memoized styles

## Naming Conventions

| Type              | Pattern                  | Example                            |
| ----------------- | ------------------------ | ---------------------------------- |
| Directories       | camelCase                | `collection`, `collectionGroup`    |
| Screen Components | PascalCase + `Screen`    | `CollectionScreen.tsx`             |
| ViewModels        | `use` + PascalCase       | `useCollectionScreenViewModel.tsx` |
| Styles            | camelCase + `.styles.ts` | `collectionScreen.styles.ts`       |
| DI Files          | camelCase + `DI.ts`      | `collectionScreenDI.ts`            |
| Routes            | camelCase + `Route.ts`   | `collectionScreenRoute.ts`         |

## Navigation Route Parameters

When navigating between screens that pass parameters, **always define the route type** in NavigationScreen.tsx:

### Step 1: Define Route Params Type

In `src/features/navigation/presentation/NavigationScreen.tsx`, create a param list type for your stack:

```typescript
// Example: Collection feature with multiple screens
export type CollectionGroupStackParamList = {
  CollectionGroup: undefined; // No params needed
  Collection: {
    collectionGroupId: string; // Required param
    collectionName: string; // Required param
  };
};
```

### Step 2: Pass Params When Navigating

In your screen component, pass the typed parameters:

```typescript
const handlePressCard = useCallback(
  (item: CollectionGroupEntity) => {
    navigation.navigate('Collection', {
      collectionGroupId: item.id,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k-angama/BattleDex](https://github.com/k-angama/BattleDex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
