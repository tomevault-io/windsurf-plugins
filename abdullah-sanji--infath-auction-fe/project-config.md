---
trigger: always_on
description: This is a modern Angular application built with:
---

# Angular Project - AI Development Guidelines

## Project Overview
This is a modern Angular application built with:
- Angular 19+ (Standalone Components)
- Zoneless change detection (Signals-based)
- Server-Side Rendering (SSR)
- PrimeNG component library
- Tailwind CSS for styling
- Template-driven forms

## Core Principles
1. **Simplicity First**: Code should be simple, readable, and maintainable
2. **Separation of Concerns**: Create reusable components and services
3. **Type Safety**: No `any` types allowed - everything must be properly typed
4. **Modern Angular**: Use latest Angular 19+ syntax and patterns
5. **Test Coverage**: Every generated code must have corresponding test functions in .spec file

---

## File Structure Requirements

### Component Files
Each component MUST have four files:
```
component-name/
├── component-name.ts        # Component logic
├── component-name.html      # Template
├── component-name.scss      # Styles
└── component-name.spec.ts   # Unit tests
```

**Never** use inline templates or inline styles except for very small, trivial components.

### Page Structure
Each page should have:
```
pages/page-name/
├── page-name.ts             # Page component
├── page-name.html           # Page template
├── page-name.scss           # Page styles
├── page-name.spec.ts        # Page tests
├── services/
│   └── page-name.service.ts # Page-specific service
└── interfaces/
    └── page-name.interface.ts # Page-specific interfaces
```

---

## Angular Best Practices

### 1. State Management - Use Signals (REQUIRED)
This is a **zoneless application**. All trackable state MUST use Angular Signals.

❌ **WRONG:**
```typescript
export class MyComponent {
  count = 0;
  user: User | null = null;
}
```

✅ **CORRECT:**
```typescript
export class MyComponent {
  count = signal(0);
  user = signal<User | null>(null);

  // Computed values
  doubleCount = computed(() => this.count() * 2);
}
```

### 2. Template Syntax - Modern Angular Control Flow
Always use Angular 19+ control flow syntax.

❌ **WRONG:**
```html
<div *ngIf="isLoggedIn">Welcome</div>
<div *ngFor="let item of items">{{ item }}</div>
<div [ngSwitch]="status">
  <div *ngSwitchCase="'active'">Active</div>
</div>
```

✅ **CORRECT:**
```html
@if (isLoggedIn()) {
  <div>Welcome</div>
}

@for (item of items(); track item.id) {
  <div>{{ item }}</div>
}

@switch (status()) {
  @case ('active') {
    <div>Active</div>
  }
  @default {
    <div>Unknown</div>
  }
}
```

### 3. Dependency Injection - Use inject() Function
Use the modern `inject()` function instead of constructor injection.

❌ **WRONG:**
```typescript
export class MyComponent {
  constructor(
    private userService: UserService,
    private router: Router
  ) {}
}
```

✅ **CORRECT:**
```typescript
export class MyComponent {
  private userService = inject(UserService);
  private router = inject(Router);
}
```

### 4. Service Injection Scope
Follow proper service injection hierarchy:

**Page-Specific Services:**
```typescript
@Injectable()  // No providedIn - inject in component
export class AuctionPageService {
  // Service used only in auction page
}

// In component:
@Component({
  providers: [AuctionPageService]  // Provide here
})
export class AuctionPage {
  private auctionService = inject(AuctionPageService);
}
```

**Project-Wide Services:**
```typescript
@Injectable({
  providedIn: 'root'  // Available everywhere
})
export class AuthService {
  // Service used across the application
}
```

### 5. Type Safety - No "any" Allowed
Every variable, parameter, and return type must be properly typed.

❌ **WRONG:**
```typescript
fetchData(): any {
  return this.http.get('/api/data');
}

processItems(items: any[]) {
  // ...
}
```

✅ **CORRECT:**
```typescript
// Create interface file
interface UserData {
  id: string;
  name: string;
  email: string;
}

fetchData(): Observable<UserData[]> {
  return this.http.get<UserData[]>('/api/data');
}

processItems(items: UserData[]): void {
  // ...
}
```

**Interface Organization:**
- Create `*.interface.ts` files per page/feature
- Use descriptive interface names
- Export all interfaces for reusability

### 6. API Calls - Exception Handling Required
ALL API calls must have proper error handling.

❌ **WRONG:**
```typescript
loadUsers(): void {
  this.http.get<User[]>('/api/users').subscribe(users => {
    this.users.set(users);
  });
}
```

✅ **CORRECT:**
```typescript
loadUsers(): void {
  this.http.get<User[]>('/api/users').subscribe({
    next: (users) => {
      this.users.set(users);
      this.error.set(null);
    },
    error: (error) => {
      console.error('Failed to load users:', error);
      this.error.set('Unable to load users. Please try again.');
      this.users.set([]);
    }
  });
}

// Or with async/await:
async loadUsers(): Promise<void> {
  try {
    const users = await firstValueFrom(
      this.http.get<User[]>('/api/users')
    );
    this.users.set(users);
    this.error.set(null);
  } catch (error) {
    console.error('Failed to load users:', error);
    this.error.set('Unable to load users. Please try again.');
    this.users.set([]);
  }
}
```

### 7. Forms - Template-Driven Only
Use template-driven forms, NOT reactive forms.

❌ **WRONG:**
```typescript
loginForm = new FormGroup({
  email: new FormControl(''),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abdullah-Sanji) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
