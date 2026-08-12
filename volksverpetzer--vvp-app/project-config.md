---
trigger: always_on
description: This repository contains the official React Native mobile app for Volksverpetzer, built with Expo.
---

# GitHub Copilot Instructions for Volksverpetzer App

This repository contains the official React Native mobile app for Volksverpetzer, built with Expo.

## Technology Stack

- **Framework**: React Native 0.83 with Expo SDK 55
- **Language**: TypeScript (with relaxed strict mode)
- **Package Manager**: pnpm (v11)
- **Testing**: Jest with jest-expo preset
- **Routing**: expo-router
- **State Management**: React hooks and context
- **UI**: Native components with custom design system

## Code Style and Conventions

### General Guidelines

- Use **arrow functions** for named React components (enforced by ESLint)
- Use **2 spaces** for indentation (defined in .editorconfig)
- Follow Prettier formatting with sorted imports (@trivago/prettier-plugin-sort-imports)
- Run `pnpm lint:fix` to automatically fix linting issues
- Use TypeScript for all new files (.ts or .tsx extensions)

### Import Order

Imports are automatically sorted by `@trivago/prettier-plugin-sort-imports` on commit. Prefer path aliases (`#/*` → `src/*`, `#assets/*`, `#tests/*`) over deep relative imports.

### Naming Conventions

- **Components**: PascalCase (e.g., `SteadyButton`, `MissionPopup`)
- **Files**: Match component name for components, camelCase for utilities
- **Functions**: camelCase (e.g., `normalizeFacets`, `registerEvent`)
- **Constants**: PascalCase for config objects, UPPER_SNAKE_CASE for true constants

### Component Structure

```tsx
import { /* native imports */ } from "react-native";
import { /* expo imports */ } from "expo-*";

import { /* local imports */ } from "../../";

interface ComponentNameProperties {
  prop1: string;
  prop2?: number;
}

/**
 * Component description with JSDoc
 * @param prop1 - Description of prop1
 * @param prop2 - Description of prop2
 */
const ComponentName = (properties: ComponentNameProperties) => {
  // Component logic
  return (
    // JSX
  );
};

export default ComponentName;
```

## Project Structure

```
src/
├── app/              # Expo Router app directory
├── components/       # Reusable UI components
│   ├── animations/
│   ├── bars/
│   ├── buttons/
│   ├── counter/
│   ├── design/
│   ├── popups/
│   ├── posts/
│   ├── typography/
│   └── views/
├── constants/        # App configuration and constants
├── helpers/          # Utility functions and business logic
│   ├── network/      # API clients (ServerAPI.ts, WordPressAPI.ts) and analytics
│   ├── Stores/       # AsyncStorage persistence layer
│   ├── provider/     # React Context providers
│   └── utils/        # General utilities
├── hooks/            # Custom React hooks
├── screens/          # Full screen components
└── types/            # TypeScript type definitions

__tests__/            # Jest test files (mirrors src/ structure)
config/               # App-specific configurations (VVP vs Mimikama)
plugins/              # Custom Expo config plugins
```

## Development Workflow

### Building and Running

```bash
# Install dependencies
pnpm install

# Start development server
pnpm start

# Run on specific platform
pnpm android
pnpm ios
pnpm web

# F-Droid / FOSS prebuild (prepare:fdroid excludes proprietary modules from autolinking)
pnpm prepare:fdroid && BUILD_FOSS_ONLY=true npx expo prebuild --platform android --no-install
```

### Testing

```bash
# Run all tests
pnpm test

# Run tests with coverage
pnpm test -- --coverage

# Run specific test file
pnpm test -- path/to/test.test.ts
```

### Code Quality

```bash
# Type checking
pnpm check:ts

# Spell checking
pnpm check:spelling

# Run all checks
pnpm check

# Lint code
pnpm lint

# Auto-fix linting issues
pnpm lint:fix
```

## Testing Guidelines

- Place tests in `__tests__/` directory mirroring the `src/` structure
- Use `.test.ts` or `.test.tsx` extensions
- Use `@testing-library/react-native` for component testing
- Mock external dependencies appropriately (see `__tests__/mocks/`)
- Follow existing test patterns in the repository

Example test structure:

```typescript
import { describe, expect, it } from "@jest/globals";

describe("ComponentName", () => {
  it("should do something", () => {
    // Arrange
    // Act
    // Assert
    expect(result).toBe(expected);
  });
});
```

## API and Data Fetching

- Helper functions for fetching are in `src/screens/Home/fetchers/`
- Analytics functions are in `src/helpers/network/Analytics.ts` (and related files in `src/helpers/network/`)
- Feed fetchers (WordPress, Instagram, TikTok, YouTube, Bluesky, etc.) are in `src/screens/Home/fetchers/`
- All fetchers return normalized Post objects

## Security

- **Gitleaks** is configured to prevent secret commits
- Never commit API keys, tokens, or sensitive credentials
- Use environment variables for configuration
- The CI pipeline includes security checks

## Multi-App Configuration

This repository supports two app variants:

- **Volksverpetzer** (default): `APP=volksverpetzer`
- **Mimikama**: `APP=mimikama`

Configuration files are in `config/` directory and are selected based on `APP` environment variable.

## Common Tasks

### Adding a New Component

1. Create component file in appropriate `src/components/` subdirectory
2. Use arrow function syntax with TypeScript interface for props
3. Add JSDoc comments describing the component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Volksverpetzer/vvp_app](https://github.com/Volksverpetzer/vvp_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
