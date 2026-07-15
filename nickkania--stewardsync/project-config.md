---
trigger: always_on
description: - **Use bun instead of npm for all commands**
---

# Agent Development Guidelines

## Essential Rules (STRICTLY FOLLOW THESE)

- **Use bun instead of npm for all commands**
- **Use node 20 through node version manager by running `nvm use 20`**
- **Do not run build commands** - I will tell you if there are build issues

---

## Project Overview

StewardSync is a unified application for reviewing racing steward reports. The application enables drivers to file incident reports, stewards to review them, and head stewards/event managers to finalize rulings.

### Technology Stack

- **Frontend:** Angular 17+ (standalone components, signals)
- **Backend:** Convex (real-time backend-as-a-service)
- **Authentication:** Convex Auth with Google OAuth 2.0
- **Styling:** Tailwind CSS 3.x
- **Package Manager:** bun
- **Node Version:** 20 (via nvm)

---

## Project Structure

```
StewardSync/
├── convex/              # Convex backend
│   ├── _generated/      # Auto-generated types
│   ├── lib/             # Shared utilities (auth, audit, errors, formatting)
│   ├── schema.ts        # Database schema definition
│   ├── auth.ts          # Authentication configuration
│   ├── users.ts         # User queries/mutations
│   ├── drivers.ts       # Driver queries/mutations
│   ├── events.ts        # Event queries/mutations
│   ├── races.ts         # Race queries/mutations
│   ├── reports.ts       # Report queries/mutations
│   ├── reviews.ts       # Review queries/mutations
│   ├── penalties.ts     # Penalty definition CRUD
│   ├── series.ts        # Series CRUD
│   ├── seriesPenalties.ts        # Series penalty definitions
│   ├── seriesPenaltyThresholds.ts # Penalty thresholds
│   ├── driverSeriesPenalties.ts   # Assigned penalties to drivers
│   ├── driverClasses.ts          # Driver classification per series
│   ├── debug.ts         # Debug/diagnostic queries (gated behind league_manager)
│   └── seed.ts          # Data seeding functions
├── src/
│   ├── app/
│   │   ├── core/        # Core services, guards, models
│   │   ├── shared/      # Reusable components, directives, pipes
│   │   ├── features/    # Feature modules (auth, reports, reviews, etc.)
│   │   └── layout/      # Layout components (header, sidebar)
│   ├── environments/    # Environment configs (dev, local, prod)
│   ├── index.html
│   ├── main.ts
│   └── styles.css       # Global styles
├── angular.json         # Angular CLI configuration
├── tailwind.config.js   # Tailwind CSS configuration
├── tsconfig.json        # TypeScript configuration
└── package.json         # Dependencies and scripts
```

---

## Data Models

### Report
- **Fields:** reportingDriver, reportedDriver, event, race, turn, description, isFinalized
- **Status Flow:** Pending → Reviewed → Finalized
- **Access:** All users can create; Stewards can edit details during review

### Review
- **Fields:** userId, reportId, incidentDescription, reviewNotes
- **Purpose:** Links stewards to reports with their assessments
- **Access:** Steward, Head Steward, Event Manager only

### User
- **Fields:** userName, role
- **Roles:** Driver, Steward, Head Steward, Event Manager

### Driver
- **Fields:** driverNumber, driverName, externalId, driverClass

### Event & Race
- **Event:** series, eventNumber, trackName, eventDate
- **Race:** eventId, raceNumber (each event can have multiple races)

---

## Angular Development Guidelines

### Components
- Use **standalone components** exclusively (no NgModule)
- Implement **signals** for reactive state management
- Follow **one component per file** convention
- Keep components small and focused (single responsibility principle)
- Use proper TypeScript types for all inputs/outputs

```typescript
import { Component, input, output } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-feature-name',
  standalone: true,
  imports: [CommonModule],
  template: `...`,
  styles: ``,
})
export class FeatureNameComponent {
  // Use signals for inputs
  readonly data = input.required<T>();
  
  // Use outputs for events
  readonly action = output<void>();
}
```

### Services
- Use Angular dependency injection
- Separate business logic from component logic
- Use `ConvexService` for Convex API calls (wraps `ConvexClient`)
- Implement proper error handling with try-catch
- Use the `inject()` function for dependency injection

```typescript
import { Injectable, inject } from '@angular/core';
import { ConvexService } from '@core/services/convex.service';

@Injectable({
  providedIn: 'root',
})
export class FeatureService {
  private readonly convex = inject(ConvexService);
  
  // Implementation
}
```

### Routing & Guards
- Use `AuthGuard` to protect authenticated routes
- Use `RoleGuard` for role-based access control
- Configure route guards in `app.routes.ts`
- Use lazy loading for feature modules

### TypeScript Best Practices
- Strict mode is enabled (enforced)
- Use interfaces for type definitions in `core/models/`
- Leverage generics where appropriate
- Avoid `any` type - use proper typing or `unknown` when necessary
- Use `readonly` modifier for inputs and immutable data

---

## Convex Backend Guidelines

### Schema Definition
- Define all tables in `convex/schema.ts`
- Use proper index configuration for frequently queried fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NickKania/StewardSync](https://github.com/NickKania/StewardSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
