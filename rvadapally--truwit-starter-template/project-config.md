---
trigger: always_on
description: Here's a comprehensive `.cursorrules` file for your Angular + .NET application that follows best practices for modularity, extensibility, and maintainability:
---

Here's a comprehensive `.cursorrules` file for your Angular + .NET application that follows best practices for modularity, extensibility, and maintainability:

```markdown
# Cursor Rules for Angular + .NET Application

## Project Overview
This is a full-stack application with Angular frontend and .NET backend services. Follow these rules to maintain consistency, quality, and best practices across the codebase.

---

## General Principles

### Code Quality
- Write clean, readable, and self-documenting code
- Follow SOLID principles
- Apply DRY (Don't Repeat Yourself) principle
- Use meaningful names for variables, methods, and classes
- Keep functions small and focused on a single responsibility
- Prefer composition over inheritance
- Write code that is testable and maintainable

### Architecture
- Maintain clear separation of concerns
- Follow modular architecture patterns
- Keep business logic separate from presentation logic
- Use dependency injection consistently
- Implement proper error handling and logging
- Design for scalability and extensibility

---

## Angular Frontend Rules

### Project Structure
```
src/
├── app/
│   ├── core/                 # Singleton services, guards, interceptors
│   │   ├── services/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── models/
│   ├── shared/               # Shared components, directives, pipes
│   │   ├── components/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── utils/
│   ├── features/             # Feature modules (lazy-loaded)
│   │   └── feature-name/
│   │       ├── components/
│   │       ├── services/
│   │       ├── models/
│   │       └── feature-name.module.ts
│   ├── layout/               # Layout components (header, footer, sidebar)
│   └── app.config.ts
├── assets/
├── environments/
└── styles/
```

### Module Organization
- Create feature modules for distinct functionality
- Implement lazy loading for feature modules
- Keep CoreModule for singleton services (import only in AppModule)
- Keep SharedModule for reusable components (import in feature modules)
- Never import SharedModule in CoreModule
- Use standalone components for new development when appropriate

### Component Best Practices
- Use OnPush change detection strategy by default
- Implement OnDestroy and unsubscribe from observables
- Keep components focused on presentation logic
- Delegate business logic to services
- Use smart/container and dumb/presentational component pattern
- Limit template complexity; use pipes and computed properties
- Use trackBy functions in *ngFor loops
- Prefer async pipe over manual subscriptions

```typescript
// Good: Smart Component
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [CommonModule, UserCardComponent]
})
export class UserListComponent implements OnInit, OnDestroy {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
  
  trackByUserId(index: number, user: User): number {
    return user.id;
  }
}

// Good: Dumb Component
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true
})
export class UserCardComponent {
  @Input({ required: true }) user!: User;
  @Output() userSelected = new EventEmitter<User>();
}
```

### Service Best Practices
- Create services for business logic and data management
- Use providedIn: 'root' for singleton services
- Implement proper error handling
- Return Observables from HTTP calls
- Use RxJS operators effectively
- Create facade services for complex state management

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly apiUrl = environment.apiUrl;
  private usersSubject = new BehaviorSubject<User[]>([]);
  
  users$ = this.usersSubject.asObservable();
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`).pipe(
      tap(users => this.usersSubject.next(users)),
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    // Log error and return user-friendly message
    return throwError(() => new Error('Failed to fetch users'));
  }
}
```

### State Management
- Use signals for reactive state (Angular 16+)
- Consider NgRx or Akita for complex state management
- Keep state immutable
- Use selectors for derived state
- Implement optimistic updates where appropriate

### HTTP and API Communication
- Create a base HTTP service with common functionality
- Use interceptors for authentication, error handling, and logging
- Implement retry logic for failed requests
- Type all HTTP responses
- Use environment files for API URLs

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }
    
    return next.handle(req);
  }
}
```

### Routing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rvadapally) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
