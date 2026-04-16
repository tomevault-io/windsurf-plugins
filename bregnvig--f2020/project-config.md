---
trigger: always_on
description: F2020 is a Formula 1 betting application built with Angular and Firebase. The application allows users to place bets on F1 races and tracks live results using OpenF1 API data.
---

# F2020 Project Context for Gemini

## Project Overview

F2020 is a Formula 1 betting application built with Angular and Firebase. The application allows users to place bets on F1 races and tracks live results using OpenF1 API data.

## Tech Stack

- **Frontend**: Angular 19, Angular Material, TailwindCSS
- **Backend**: Firebase (Firestore, Functions, Cloud Messaging)
- **Build System**: Nx monorepo
- **APIs**: OpenF1 for live F1 data

## Key Commands

### Development

- `npm run start` or `nx serve ui` - Start UI development server
- `nx serve builder` - Run data builder (populate Firestore)
- `npm run serve:firebase` - Start Firebase functions development

### Testing & Quality

- `npm test` - Run tests
- `npm run lint` - Run linting
- `npm run format` - Format code

### Firebase/Emulator

- `firebase emulators:start --only=functions,firestore,auth,pubsub --config=firebase.json --export-on-exit=./saved-data --import=./saved-data --inspect-functions --project f1-playground-e1f23`
- `npm run kill-ports` - Kill occupied ports

### Build & Deploy

- `npm run build:ui` - Build UI for production
- `npm run deploy:all` - Deploy hosting, functions, and rules
- `npm run deploy:hosting` - Deploy hosting only
- `npm run deploy:functions` - Deploy functions only

## Project Structure

- `apps/ui/` - Angular frontend application
- `apps/functions/` - Firebase Cloud Functions
- `apps/builder/` - Data import/migration utilities
- `libs/` - Shared libraries and feature modules
  - `libs/api/` - API services and data layer
  - `libs/data/` - Data models and mappers
  - `libs/openf1/` - OpenF1 API types
  - `libs/shared/` - Shared UI components
  - Other feature-specific libraries (race, player, standing, etc.)

## Key Services

- `LiveResultService` - Handles live race data and WebSocket connections
- `OpenF1HttpService` - HTTP client for OpenF1 API
- `OpenF1WSSService` - WebSocket service for real-time F1 data

## Development Notes

- Uses Nx for monorepo management
- Firebase emulator for local development
- OpenF1 API integration for live race data
- WebSocket connections for real-time updates
- Angular standalone components pattern

## TypeScript & Functional Programming Style

This project emphasizes a modern, functional, and idiomatic TypeScript style. We aim to write code that is not only correct but also clean, expressive, and leverages the full power of the language. Avoid writing TypeScript as if it were a class-based language like Java or C#.

### Embrace Functional Constructs

- **Use `map`, `reduce`, `filter`**: Prefer functional array methods over imperative loops (`for`, `while`). This leads to more declarative and readable code.

  ```typescript
  // ✅ PREFERRED: Functional and declarative
  const numbers = [1, 2, 3, 4];
  const doubledAndSummed = numbers
    .map(n => n * 2)
    .reduce((sum, n) => sum + n, 0);

  // ❌ AVOID: Imperative loop
  let sum = 0;
  for (const n of numbers) {
    sum += n * 2;
  }
  ```

- **Immutability**: Treat data as immutable where possible. Instead of modifying objects and arrays directly, create new ones with the updated values.

  ```typescript
  // ✅ PREFERRED: Immutable update
  const user = { id: 1, name: 'John' };
  const updatedUser = { ...user, name: 'Jane' };

  // ❌ AVOID: Direct mutation
  const userToUpdate = { id: 1, name: 'John' };
  userToUpdate.name = 'Jane'; // Mutation
  ```

### Leverage Modern ESNext Features

- **Optional Chaining (`?.`) and Nullish Coalescing (`??`)**: Use these operators to handle potentially null or undefined values gracefully and concisely.

  ```typescript
  // ✅ PREFERRED: Clean and safe
  const userName = user?.profile?.name ?? 'Guest';

  // ❌ AVOID: Verbose and error-prone checks
  let userNameResult = 'Guest';
  if (user && user.profile && user.profile.name) {
    userNameResult = user.profile.name;
  }
  ```

### Write "Real" TypeScript

- **Strong Typing**: Leverage TypeScript's type system. Avoid `any` whenever possible. Define clear interfaces and types for your data structures.
- **Code with a TypeScript Mindset**: Don't just write JavaScript and add types later. Think in terms of types from the beginning. This helps in designing better APIs and components.
- **Keep it Lean**: Avoid overly complex class hierarchies or design patterns borrowed from languages like Java/C# if a simpler functional approach will suffice. TypeScript is a multi-paradigm language; use the right tool for the job.

## Angular Coding Standards

### Functional API (Preferred)

**Always use Angular's functional API instead of decorators**:

```typescript
import { booleanAttribute } from '@angular/core';

// ✅ PREFERRED: Functional API
export class MyComponent {
  placeholder = input<string>();
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });
  valueChange = output<string>();
}

// ❌ AVOID: Decorator API
export class MyComponentWithDecorators {
  @Input() placeholder?: string;
  @Input() disabled = false;
  @Output() valueChange = new EventEmitter<string>();
}
```

**Benefits of Functional API**:

- Better type inference
- Easier to test and mock
- More consistent with modern Angular patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bregnvig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
