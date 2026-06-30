---
trigger: always_on
description: This is an enterprise-grade admin dashboard built with Angular 21, Angular Material 3, and Tailwind CSS 4. The application follows modern Angular best practices with standalone components, signals-based state management, and a comprehensive design system that supports responsive layouts and light/dark theming.
---

# GitHub Copilot Custom Instructions: Angular 21 + Material 3 + Tailwind 4 Enterprise Admin Dashboard

## Project Overview
This is an enterprise-grade admin dashboard built with Angular 21, Angular Material 3, and Tailwind CSS 4. The application follows modern Angular best practices with standalone components, signals-based state management, and a comprehensive design system that supports responsive layouts and light/dark theming.

---

## CRITICAL RULES - READ FIRST

### 1. Technology Stack (Non-Negotiable)
- **Angular 21**: Use latest features (signals, new control flow, standalone components)
- **Angular Material 19+**: Material Design 3 components only
- **Tailwind CSS 4**: CSS-first configuration (@theme syntax, NO tailwind.config.js)
- **TypeScript 5.5+**: Strict mode enabled

### 2. Framework Hierarchy
```
ALWAYS FOLLOW THIS ORDER:
1. Angular Material → Structure, behavior, accessibility
2. Tailwind 4 → Layout, spacing, responsive utilities
3. Custom CSS → ONLY when absolutely necessary

NEVER override Material component internals with custom CSS.
NEVER duplicate Material functionality with Tailwind.
```

---

## ANGULAR 21 REQUIREMENTS

### Component Architecture

#### ALWAYS Use Standalone Components
```typescript
// ✅ CORRECT
@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule, MatCardModule, ...],
  template: `...`
})
export class DashboardComponent {}

// ❌ WRONG - No NgModule components
@NgModule({ ... })
```

#### ALWAYS Use Signals for State Management
```typescript
// ✅ CORRECT - Use signals
export class MyComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);
  items = signal<Item[]>([]);
  
  increment() {
    this.count.update(n => n + 1);
  }
}

// ❌ WRONG - Don't use traditional properties
export class MyComponent {
  count = 0;  // AVOID
  items: Item[] = [];  // AVOID
}
```

#### ALWAYS Use New Control Flow Syntax
```typescript
// ✅ CORRECT - New @syntax
@Component({
  template: `
    @if (isLoggedIn()) {
      <app-dashboard />
    } @else {
      <app-login />
    }
    
    @for (item of items(); track item.id) {
      <app-item [data]="item" />
    }
    
    @switch (status()) {
      @case ('loading') { <app-loader /> }
      @case ('success') { <app-content /> }
      @case ('error') { <app-error /> }
    }
  `
})

// ❌ WRONG - Old *ng syntax
template: `
  <div *ngIf="isLoggedIn">...</div>  // NEVER USE
  <div *ngFor="let item of items">...</div>  // NEVER USE
  <div [ngSwitch]="status">...</div>  // NEVER USE
`
```

#### ALWAYS Use Input Signals (Angular 17.2+)
```typescript
// ✅ CORRECT - Input signals
export class UserCard {
  userId = input.required<string>();
  userName = input<string>('Guest');
  isActive = input<boolean>(false);
  
  // Computed from inputs
  displayName = computed(() => 
    `${this.userName()} (${this.isActive() ? 'Active' : 'Inactive'})`
  );
}

// ❌ WRONG - Old @Input decorator
export class UserCard {
  @Input() userId!: string;  // AVOID
  @Input() userName = 'Guest';  // AVOID
}
```

#### ALWAYS Use toSignal for Observable Conversion
```typescript
// ✅ CORRECT - Convert observables to signals
export class MyComponent {
  private breakpointObserver = inject(BreakpointObserver);
  
  isMobile = toSignal(
    this.breakpointObserver.observe([Breakpoints.Handset]),
    { initialValue: false }
  );
}

// ❌ WRONG - Subscribe in component
ngOnInit() {
  this.breakpointObserver.observe(...).subscribe(...);  // AVOID
}
```

#### ALWAYS Use OnPush Change Detection
```typescript
// ✅ CORRECT
@Component({
  selector: 'app-my-component',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,  // ALWAYS ADD
  template: `...`
})
```

#### ALWAYS Use inject() Function (Angular 14+)
```typescript
// ✅ CORRECT - inject() in class body
export class MyComponent {
  private router = inject(Router);
  private fb = inject(FormBuilder);
  private myService = inject(MyService);
}

// ❌ WRONG - Constructor injection (legacy style)
constructor(
  private router: Router,
  private fb: FormBuilder
) {}  // AVOID unless necessary
```

---

## TAILWIND 4 CONFIGURATION

### CRITICAL: Tailwind 4 Uses CSS-First Configuration

#### File Structure
```
src/
├── styles/
│   ├── tailwind.css         ← Main Tailwind file with @theme
│   ├── _material-theme.scss ← Material theming
│   └── _utilities.css       ← Custom utilities
└── styles.scss              ← Global styles
```

#### ALWAYS Use @theme in tailwind.css
```css
/* src/styles/tailwind.css */
@import "tailwindcss";

@theme {
  /* CRITICAL: Prefix prevents Material conflicts */
  --prefix: tw;
  
  /* Spacing: 8px grid system */
  --spacing-0: 0;
  --spacing-xs: 0.25rem;   /* 4px */
  --spacing-sm: 0.5rem;    /* 8px - Material base */
  --spacing-md: 1rem;      /* 16px */
  --spacing-lg: 1.5rem;    /* 24px */
  --spacing-xl: 2rem;      /* 32px */
  --spacing-2xl: 3rem;     /* 48px */
  --spacing-3xl: 4rem;     /* 64px */
  
  /* Colors: Sync with Material palette */
  --color-primary-50: #e8eaf6;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nitin27may/clean-architecture-docker-dotnet-angular](https://github.com/nitin27may/clean-architecture-docker-dotnet-angular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
