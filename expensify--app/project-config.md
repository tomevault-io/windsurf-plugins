---
trigger: always_on
description: - **Framework**: React Native
---

# Expensify App

## Repository Overview

### Technology Stack

- **Framework**: React Native
- **Language**: TypeScript
- **State Management**: React Native Onyx
- **Navigation**: React Navigation
- **Platforms**: iOS, Android, Web

## HybridApp Architecture (Critical Context)

**IMPORTANT**: The mobile application is built from the Mobile-Expensify submodule, not directly from the App repository.
**IMPORTANT**: NewDot refers to the new Expensify App, OldDot or Expensify Classic refers to our Old expensify app and website

### Key Integration Points

- App (NewDot) and Mobile-Expensify (OldDot) are combined into a single mobile application
- The HybridApp module (`@expensify/react-native-hybrid-app`) manages transitions between OldDot and NewDot
- Build process merges dotenv configurations from both repositories
- Environment variables from Mobile-Expensify take precedence over App variables
- Mobile builds **must** be initiated from the Mobile-Expensify directory

### Build Modes

- **Standalone**: Pure NewDot application (web)
- **HybridApp**: Combined OldDot + NewDot (mobile apps)
- Controlled via `STANDALONE_NEW_DOT` environment variable

## Core Architecture & Structure

### Entry Points

- `src/App.tsx`: Main application component with provider hierarchy
- `src/Expensify.tsx`: Core application logic and initialization
- `src/HybridAppHandler.tsx`: Manages HybridApp transitions and authentication
- `index.js`: React Native entry point

### Provider Architecture

The application uses a nested provider structure for context management:

1. **SplashScreenStateContextProvider**: Manages splash screen visibility
2. **InitialURLContextProvider**: Handles deep linking
3. **ThemeProvider**: Theme management
4. **LocaleContextProvider**: Internationalization
5. **OnyxListItemProvider**: Data layer provider
6. **SafeAreaProvider**: Device safe areas
7. **PopoverContextProvider**: Global popover state
8. **KeyboardProvider**: Keyboard state management

### Data Layer

- **Onyx**: Custom data persistence layer for offline-first functionality
- **ONYXKEYS.ts**: Centralized key definitions for data store
- Supports optimistic updates and conflict resolution

## Key Features & Modules

### Core Functionality

1. **Expense Management**
   - Receipt scanning and SmartScan
   - Expense creation and editing
   - Distance tracking
   - Per diem support
   - Split expenses

2. **Reporting**
   - Report creation and submission
   - Approval workflows
   - Report fields and custom attributes
   - Bulk operations

3. **Workspace/Policy Management**
   - Policy creation and configuration
   - Member management
   - Categories, tags, and tax rates
   - Accounting integration settings
   - Approval workflows

4. **Travel**
   - Trip management
   - Travel booking integration
   - Travel policy enforcement

5. **Search & Filtering**
   - Advanced search with filters
   - Saved searches
   - Search parser (Peggy-based)

6. **Payment & Cards**
   - Expensify Card management
   - Bank account connections (Plaid)
   - Payment methods
   - Company cards integration
   - Wallet functionality

7. **Accounting Integrations**
   - QuickBooks Online
   - Xero
   - NetSuite
   - Sage Intacct
   - QuickBooks Desktop
   - Generic accounting connections

8. **Communication**
   - Chat functionality
   - Task management
   - Mentions and notifications
   - Thread organization

9. **Invoice Management**
   - Invoice creation and sending
   - Invoice rooms

## Navigation & Routing

### Structure

- `src/SCREENS.ts`: Screen name constants
- `src/ROUTES.ts`: Route definitions and builders
- `src/NAVIGATORS.ts`: Navigator configuration

### Key Navigators

- **ProtectedScreens**: Authenticated app screens
- **PublicScreens**: Login and onboarding screens
- **RHP (Right Hand Panel/Pane)**: Settings and details panel
- **Central Pane**: Main content area
- **LHN (Left Hand Navigation)**: Report list and navigation
- **RHP**: Contextual panels and settings

## State Management

### Onyx Keys Organization

- **Session**: Authentication and user session
- **Personal Details**: User profiles and preferences
- **Reports**: Chat and expense reports
- **Transactions**: Expense transactions
- **Policy**: Workspace configuration
- **Forms**: Form state management

### Action Modules (`src/libs/actions/`)

Major action categories:

- `App.ts`: Application lifecycle
- `IOU.ts`: Money requests and expenses
- `Report.ts`: Report management
- `Policy/`: Workspace operations
- `User.ts`: User account operations
- `Session.ts`: Authentication
- `Search.ts`: Search operations
- `Travel.ts`: Travel features

## Build & Deployment

### CI/CD Workflows

Key GitHub Actions workflows:

- `deploy.yml`: Production deployment
- `preDeploy.yml`: Staging deployment
- `testBuild.yml`: PR test builds
- `test.yml`: Unit tests
- `typecheck.yml`: TypeScript validation
- `lint.yml`: Code quality checks

## Related Repositories

### Mobile-Expensify (Submodule)

- **Path**: `App/Mobile-Expensify/`
- **Purpose**: Legacy OldDot application and mobile build source
- **Critical**: All mobile builds originate from this directory
- Contains platform-specific code for iOS and Android
- Manages the HybridApp integration layer

### expensify-common

- **Purpose**: Shared libraries and utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Expensify/App](https://github.com/Expensify/App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
