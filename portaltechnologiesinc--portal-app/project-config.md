---
trigger: always_on
description: - **Structured Communication**: The user prefers receiving instructions as structured, step-by-step plans of action, with separate tasks when executing changes, rather than unstructured explanations or code snippets, as this approach yields better results.
---

# Portal App - Cursor AI Rules

## AI Assistant Guidelines

### User Interaction Preferences

- **Structured Communication**: The user prefers receiving instructions as structured, step-by-step plans of action, with separate tasks when executing changes, rather than unstructured explanations or code snippets, as this approach yields better results.

### Development Approach

- **Minimal Changes**: Think carefully and only action the specific task I have given you with the most concise and elegant solution that changes as little code as possible. Do not override changes made by the user.

## Project Overview

Portal is a React Native mobile identity wallet built with Expo for Nostr protocol authentication and Lightning payments. This is a security-focused application with enterprise-grade requirements.

## Technology Stack

- **Framework**: React Native 0.79.5 + Expo 53.0.19
- **Language**: TypeScript 5.8.3 (strict mode enabled)
- **Navigation**: Expo Router (file-based routing)
- **Database**: Expo SQLite with custom service layer
- **State Management**: React Context (multiple specialized contexts)
- **Styling**: Custom theme system with dark/light mode support
- **Icons**: Lucide React Native
- **Security**: Expo SecureStore, LocalAuthentication (biometrics)

## Code Style & Formatting

### Prettier Configuration

- Semi-colons: required (`;`)
- Quotes: single quotes (`'`)
- Trailing commas: ES5 style
- Print width: 100 characters
- Tab width: 2 spaces
- No tabs (spaces only)
- Bracket spacing: enabled
- Arrow function parens: avoid when possible

### ESLint Rules

- Follow TypeScript recommended rules
- React recommended rules + hooks rules
- No `React` import required (React 19)
- Allow `require()` imports for React Native context
- Prettier integration enforced as errors

## Architecture Patterns

### Directory Structure

```
app/                    # Expo Router pages (file-based routing)
├── (tabs)/            # Tab navigation screens
├── activity/[id]/     # Dynamic routes for activity details
└── subscription/[id]/ # Dynamic routes for subscriptions

components/            # Reusable UI components
├── ActivityDetail/   # Feature-specific components
└── ui/              # Base UI components

context/              # React Context providers (state management)
services/             # Business logic and external integrations
models/              # TypeScript interfaces and types
utils/               # Helper functions and utilities
constants/           # App configuration and constants
hooks/               # Custom React hooks
```

### State Management

- Use React Context for state management, NOT Redux or Zustand
- Create specialized contexts for different domains (Activities, Nostr, Theme, etc.)
- Access database through `useSQLiteContext()` and `DatabaseService` class
- Use secure storage for sensitive data via `SecureStorageService`

### Database Operations

```typescript
// Always use the DatabaseService pattern
const db = useSQLiteContext();
const dbService = new DatabaseService(db);
```

### Theme-Aware Components

```typescript
// All components must support theme system
import { useThemeColor } from '@/hooks/useThemeColor';

const backgroundColor = useThemeColor({}, 'background');
const textColor = useThemeColor({}, 'textPrimary');
```

## Security Requirements

### Critical Security Rules

1. **Private keys NEVER leave the device** - use SecureStore only
2. **Biometric authentication required** for sensitive operations
3. **All user inputs must be validated** and sanitized
4. **No sensitive data in logs** or console outputs
5. **Use HTTPS only** for all network requests
6. **Implement proper error handling** without exposing internal details

### Biometric Authentication Pattern

```typescript
import { BiometricAuthService } from '@/services/BiometricAuthService';

// Always verify biometric capability before sensitive operations
if (await BiometricAuthService.isAvailable()) {
  const authResult = await BiometricAuthService.authenticate();
  if (authResult.success) {
    // Proceed with sensitive operation
  }
}
```

## Component Development

### React Native Best Practices

- Use functional components with hooks (no class components)
- Implement proper keyboard handling with `KeyboardAvoidingView`
- Use `SafeAreaView` or safe area hooks for proper device rendering
- Handle platform differences with `Platform.OS` when necessary
- Use proper React Native navigation patterns with Expo Router

### UI Component Guidelines

- All components must support both light and dark themes
- Use Lucide React Native for icons consistently
- Implement proper accessibility labels and hints
- Use consistent spacing and typography from theme system
- Follow platform-specific design guidelines (iOS/Android)

### Performance Considerations

- Use `React.memo()` for expensive components
- Implement proper list virtualization for large datasets
- Optimize images and assets appropriately
- Use lazy loading for heavy screens
- Minimize bundle size by avoiding unnecessary dependencies

## Nostr Protocol Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PortalTechnologiesInc/portal-app](https://github.com/PortalTechnologiesInc/portal-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
