---
trigger: always_on
description: Guidelines for developing the Armature documentation website in the `web/` directory.
---


# Angular Documentation Website

Guidelines for developing the Armature documentation website in the `web/` directory.

## Technology Stack

- **Framework:** Angular 21 (standalone components)
- **Package Manager:** pnpm (required)
- **Testing:** Vitest
- **Styling:** CSS/SCSS

## Project Structure

```
web/
├── src/
│   ├── app/
│   │   ├── components/     # Shared UI components
│   │   ├── pages/          # Route pages
│   │   ├── services/       # Angular services
│   │   ├── models/         # TypeScript interfaces
│   │   └── app.component.ts
│   ├── assets/             # Static assets
│   └── styles/             # Global styles
├── public/                 # Public static files
├── angular.json
├── package.json
├── vitest.config.ts
└── tsconfig.json
```

## Angular 21 Patterns

### Standalone Components (Required)

All components must be standalone:

```typescript
// ✅ Good: Standalone component
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `...`,
})
export class FeatureComponent { }

// ❌ Bad: Non-standalone (deprecated pattern)
@Component({
  selector: 'app-feature',
  template: `...`,
})
export class FeatureComponent { }
// Then added to NgModule declarations
```

### Signals (Preferred for State)

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <button (click)="increment()">Count: {{ count() }}</button>
    <p>Double: {{ doubleCount() }}</p>
  `,
})
export class CounterComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);

  constructor() {
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }

  increment() {
    this.count.update(c => c + 1);
  }
}
```

### New Control Flow Syntax

```typescript
// ✅ Good: New control flow (Angular 17+)
@Component({
  template: `
    @if (isLoading()) {
      <app-spinner />
    } @else if (error()) {
      <app-error [message]="error()" />
    } @else {
      @for (item of items(); track item.id) {
        <app-item [data]="item" />
      } @empty {
        <p>No items found</p>
      }
    }

    @switch (status()) {
      @case ('pending') { <span>Pending...</span> }
      @case ('success') { <span>Success!</span> }
      @default { <span>Unknown</span> }
    }
  `,
})
export class ListComponent {
  items = signal<Item[]>([]);
  isLoading = signal(false);
  error = signal<string | null>(null);
  status = signal<'pending' | 'success' | 'error'>('pending');
}

// ❌ Bad: Old structural directives
@Component({
  template: `
    <ng-container *ngIf="isLoading; else loaded">
      <app-spinner></app-spinner>
    </ng-container>
    <ng-template #loaded>
      <app-item *ngFor="let item of items; trackBy: trackById" [data]="item"></app-item>
    </ng-template>
  `,
})
```

### Inject Function (Preferred)

```typescript
// ✅ Good: inject() function
@Component({
  selector: 'app-feature',
  standalone: true,
  template: `...`,
})
export class FeatureComponent {
  private http = inject(HttpClient);
  private route = inject(ActivatedRoute);
  private docsService = inject(DocsService);
}

// ❌ Less preferred: Constructor injection
@Component({
  selector: 'app-feature',
  standalone: true,
  template: `...`,
})
export class FeatureComponent {
  constructor(
    private http: HttpClient,
    private route: ActivatedRoute,
    private docsService: DocsService,
  ) { }
}
```

## Commands

```bash
# Development server
cd web
pnpm start          # Starts on http://localhost:4200

# Build for production
pnpm run build

# Run tests
pnpm test           # Uses Vitest

# Lint
pnpm run lint
```

## Documentation Content Integration

### Loading Markdown Docs

The website should load and render markdown documentation from `docs/`:

```typescript
@Injectable({ providedIn: 'root' })
export class DocsService {
  private http = inject(HttpClient);

  getDoc(slug: string): Observable<string> {
    return this.http.get(`/docs/${slug}.md`, { responseType: 'text' });
  }
}
```

### Code Syntax Highlighting

Use Prism.js or highlight.js for Rust code highlighting:

```typescript
@Component({
  selector: 'app-code-block',
  standalone: true,
  template: `
    <pre><code [innerHTML]="highlightedCode()"></code></pre>
  `,
})
export class CodeBlockComponent {
  code = input.required<string>();
  language = input<string>('rust');

  highlightedCode = computed(() => {
    return Prism.highlight(
      this.code(),
      Prism.languages[this.language()],
      this.language()
    );
  });
}
```

## Styling Guidelines

### CSS Variables for Theming

```css
:root {
  /* Colors */
  --color-primary: #ff6b35;
  --color-secondary: #3498db;
  --color-background: #1a1a2e;
  --color-surface: #16213e;
  --color-text: #eaeaea;
  --color-text-muted: #888888;

  /* Typography */
  --font-heading: 'JetBrains Mono', monospace;
  --font-body: 'Inter', sans-serif;
  --font-code: 'Fira Code', monospace;

  /* Spacing */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --space-xl: 4rem;
}
```

### Responsive Design

```css
/* Mobile-first approach */
.container {
  padding: var(--space-md);
}

@media (min-width: 768px) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
