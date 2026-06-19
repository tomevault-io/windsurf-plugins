---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProductivityPlanner is an Angular 20+ webapp for personal productivity using the Pomodoro technique. It uses Firebase for authentication and hosting, with NgRx Signals for state management.

## Development Commands

### Basic Commands
- `npm install` - Install dependencies
- `npm run start:development` - Start development server
- `npm run start:staging` - Start staging server  
- `npm run start:production` - Start production server

### Build Commands
- `npm run build:development` - Development build
- `npm run build:staging` - Staging build
- `npm run build:production` - Production build
- `npm run watch` - Watch mode for development

### Testing
- `npm run test` - Run Jest unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report
- `npm run test:ci` - Run tests for CI
- `npm run e2e:dev` - Run Playwright e2e tests with UI
- `npm run e2e:ci` - Run e2e tests for CI

### Code Quality
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier

### Storybook
- `npm run storybook` - Start Storybook dev server (port 6006)
- `npm run build-storybook` - Build Storybook static files
- `npm run test-storybook` - Test Storybook components

### Deployment
- `npm run deploy:development` - Build and deploy to Firebase development
- `npm run deploy:staging` - Build and deploy to Firebase staging  
- `npm run deploy:production` - Build and deploy to Firebase production

## Architecture

### Module Structure
The application follows a feature-based architecture with clear separation between visitor and authenticated user areas:

- **Core Module** (`src/app/core/`): Contains shared services, stores, entities, and adapters
- **Visitor Module** (`src/app/visitor/`): Public pages (home, login, signup)
- **Membership Module** (`src/app/membership/`): Authenticated user area with shell layout
- **Shared Module** (`src/app/shared/`): Reusable components across modules

### Key Architectural Patterns

#### Hexagonal Architecture
The project implements hexagonal architecture with:
- **Ports** (`src/app/core/ports/`): Abstract service interfaces
- **Adapters** (`src/app/core/adapters/`): Concrete implementations (Firebase services)
- **Domain** (`src/app/*/domain/`): Business logic and use cases within feature modules
- **Use Cases** (`src/app/*/domain/*.use-case.ts`): Application-specific business rules (e.g., `AddTaskUseCase`, `LoginUserUseCase`, `RegisterUserUseCase`)

#### Domain Entities
The application uses immutable entities for core business objects:
- **Task** (`src/app/core/entities/task.entity.ts`): Represents a task with immutable properties
- **Workday** (`src/app/core/entities/workday.entity.ts`): Represents a workday with immutable task list management
- **User** (`src/app/core/entities/user.interface.ts`): User interface for authentication state

#### Component Naming Convention
- **Smart Components**: `.smart.component.ts` - Handle business logic and state
- **Dumb Components**: `.dumb.component.ts` - Pure presentation components
- **Page Components**: `.page.component.ts` - Route-level components
- **Layout Components**: `.layout.component.ts` - Structural layout components

#### State Management
- Uses **NgRx Signals** for state management
- User store at `src/app/core/stores/user.store.ts` manages authentication state
- Provides computed properties like `isGoogleUser`

#### Authentication Flow
- Firebase Authentication via `AuthenticationFirebaseService`
- JWT tokens with refresh token support
- Error handling with custom error types (`EmailAlreadyTakenError`, `InvalidCredentialError`)

### Path Aliases
The project uses TypeScript path mapping:
- `@core/*` → `src/app/core/*`
- `@membership/*` → `src/app/membership/*`
- `@visitor/*` → `src/app/visitor/*`
- `@shared/*` → `src/app/shared/*`
- `@environments/*` → `src/environments/*`

### Routing Structure
- **Main Routes**: Home, Login, Signup in `app.routes.ts`
- **Membership Routes**: Dashboard, Planning, Workday, Profile, Settings in `membership.routes.ts`
- Uses lazy loading for membership area via `ShellLayoutComponent`

### Testing Setup
- **Jest** for unit testing with Angular preset
- **Playwright** for e2e testing
- **Storybook** for component documentation and testing
- Coverage reports generated in `coverage/` directory

#### Component Testing Best Practices

**IMPORTANT**: When writing component tests, you MUST follow these rules:

1. **Use `data-testid` attributes**: ALWAYS use `data-testid` attributes to reference DOM elements in tests
   - Add `data-testid` attributes to all interactive elements (buttons, inputs, links, etc.)
   - Use descriptive, kebab-case names (e.g., `data-testid="add-task-button"`)
   - Never use CSS classes or element selectors in component tests

2. **BDD-style test descriptions**: Write test `describe()` blocks from the user's perspective
   - Use "when user..." format (e.g., `describe('when user clicks delete button')`)
   - Focus on user behavior, not implementation details

**Example:**
```typescript
// Template

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArnoldM/productivity-planner](https://github.com/ArnoldM/productivity-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
