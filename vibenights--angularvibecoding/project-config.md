---
trigger: always_on
description: This is a modern Angular 20 application using standalone components, signals, TailwindCSS, and DaisyUI. The project is optimized for AI-assisted development and follows strict best practices.
---

# Angular 20 VibeCoding Project - Cursor Rules

## Project Overview

This is a modern Angular 20 application using standalone components, signals, TailwindCSS, and DaisyUI. The project is optimized for AI-assisted development and follows strict best practices.

## Core Technologies

- **Angular 20+** with standalone architecture (no NgModules)
- **TypeScript** with strict type checking
- **Angular Signals** for reactive state management
- **TailwindCSS 4** + **DaisyUI** for styling
- **SCSS** as the primary styling engine
- **RxJS** for complex asynchronous operations only
- **Angular Material** for additional UI components

## Critical Naming Conventions

⚠️ **IMPORTANT**: This project does NOT use standard Angular file suffixes

- ✅ Use: `login.ts`, `auth.ts`, `notification.ts`
- ❌ Never use: `login.component.ts`, `auth.service.ts`, `notification.service.ts`
- Components, services, guards, and pipes all use simple `.ts` extension
- Each feature has its own folder with `.ts`, `.html`, `.scss`, and `.spec.ts` files

## File Structure Pattern

```
feature-name/
  ├── feature-name.ts        # Component or service
  ├── feature-name.html      # Template (if component)
  ├── feature-name.scss      # Styles (if component)
  └── feature-name.spec.ts   # Tests
```

## TypeScript Best Practices

### Always Use Strict Typing

- ❌ NEVER use `any` type
- ✅ Define proper interfaces for all data structures
- ✅ Use type inference where obvious
- ✅ Use `unknown` for uncertain types (rare cases)

```typescript
// ✅ Good
interface LoginCredentials {
  email: string;
  password: string;
}

// ❌ Bad
const credentials: any = { email: "", password: "" };
```

## Angular Component Best Practices

### Component Structure

```typescript
import {
  Component,
  signal,
  computed,
  inject,
  input,
  output,
} from "@angular/core";
import { ChangeDetectionStrategy } from "@angular/core";

@Component({
  selector: "app-example",
  templateUrl: "./example.html",
  styleUrls: ["./example.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    /* standalone imports */
  ],
})
export class Example {
  // Use inject() instead of constructor injection
  private router = inject(Router);
  private authService = inject(Auth);

  // Use input() and output() functions instead of decorators
  title = input<string>("Default Title");
  users = input<User[]>([]);
  formSubmitted = output<FormData>();

  // Use signals for local state
  isLoading = signal(false);
  isActive = signal(false);

  // Use computed() for derived state
  isFormValid = computed(() => !this.nameError() && !this.emailError());
  backgroundColor = computed(() => (this.isActive() ? "#e3f2fd" : "#ffffff"));
}
```

### Key Component Rules

1. ✅ All components use `ChangeDetectionStrategy.OnPush`
2. ✅ Use `inject()` function instead of constructor injection
3. ✅ Use `input()` and `output()` functions instead of `@Input()` and `@Output()` decorators
4. ✅ Use signals for all local state
5. ✅ Use `computed()` for all derived state
6. ✅ Keep components small and focused (single responsibility)
7. ✅ All components are standalone (no NgModules)

## Template Best Practices

### Use Native Control Flow (NOT *ngIf, *ngFor, \*ngSwitch)

```html
<!-- ✅ Good: Use native control flow -->
@if (isVisible()) {
<h2>{{ title() }}</h2>
} @for (user of users(); track user.id) {
<div class="user-item">
  <span>{{ user.name }}</span>
</div>
} @switch (status()) { @case ('loading') {
<div class="loading">Loading...</div>
} @case ('success') {
<div class="success">Success!</div>
} @default {
<div class="default">Default state</div>
} }

<!-- ❌ Bad: Don't use old syntax -->
<div *ngIf="isVisible()">...</div>
<div *ngFor="let user of users()">...</div>
```

### Use Class and Style Bindings (NOT ngClass/ngStyle)

```html
<!-- ✅ Good: Use direct bindings -->
<div [class.active]="isActive()" [class.disabled]="isDisabled()">
  Dynamic classes
</div>

<div [style.background-color]="backgroundColor()" [style.color]="textColor()">
  Dynamic styles
</div>

<!-- ❌ Bad: Don't use ngClass/ngStyle -->
<div [ngClass]="{'active': isActive()}">...</div>
<div [ngStyle]="{'background-color': backgroundColor()}">...</div>
```

### Always Use Async Pipe for Observables

```html
<!-- ✅ Good -->
@if (user$ | async; as user) {
<div>{{ user.name }}</div>
}

<!-- ❌ Bad: Don't manually subscribe in templates -->
```

## Service Best Practices

### Service Structure

```typescript
import { Injectable, inject, signal } from "@angular/core";

@Injectable({
  providedIn: "root", // Always use root provider
})
export class Auth {
  // Use inject() instead of constructor injection
  private router = inject(Router);
  private notification = inject(NotificationService);

  // Use signals for service state
  currentUser = signal<User | null>(null);
  isAuthenticated = computed(() => this.currentUser() !== null);

  // Service methods...
}
```

### Key Service Rules

1. ✅ All services use `providedIn: 'root'`
2. ✅ Use `inject()` function instead of constructor injection
3. ✅ Single responsibility principle
4. ✅ Use signals for state management

## State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VibeNights/angularvibecoding](https://github.com/VibeNights/angularvibecoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
