---
trigger: always_on
description: Ask the user which tracker to use for each workflow; local Markdown under `.scratch/` is supported. See `docs/agents/issue-tracker.md`.
---

# AGENTS.md - POS System Development Guidelines

## Agent skills

### Issue tracker

Ask the user which tracker to use for each workflow; local Markdown under `.scratch/` is supported. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the standard five-role triage vocabulary. See `docs/agents/triage-labels.md`.

### Domain docs

This is a single-context repository. See `docs/agents/domain.md`.

## Build, Lint, and Test Commands

### Development
```bash
npm run dev              # Start dev server (http://localhost:5173)
npm run dev:https        # Start with HTTPS
npm run dev:network      # Start accessible on network
npm run electron:dev     # Run with Electron (React + electron)
npm run electron:dev-debug  # Run with Electron + Chrome debugging
```

### Building
```bash
npm run build            # Build for production (outputs to dist/)
npm run preview          # Preview production build locally
```

### Linting & Type Checking
```bash
npm run lint             # Run ESLint on all files
```

### Testing
```bash
npm test                 # Run all tests
npm test -- src/tests/Auth.test.tsx  # Run specific test file
npm test -- --run       # Run tests once (no watch mode)
npm test -- --coverage  # Run with coverage report
```

### End-to-end (Playwright)
```bash
npx playwright install chromium   # One-time browser download (per machine)
npm run test:e2e                  # Headless E2E (starts Vite via playwright.config)
npm run test:e2e:ui               # Playwright UI mode
npm run test:e2e:headed           # See the browser while tests run
npm run test:e2e:debug            # Step through with Playwright Inspector
```

`e2e/auth.spec.ts` signs in as the bootstrap admin from `public/bootstrap-data.json` using the demo password **`password`** (SHA-256 stored in `password_hash`). Adjust that file if you change credentials.

### Electron Distribution
```bash
npm run electron:dist           # Build for current platform
npm run electron:dist:all       # Build for all platforms
npm run electron:pack           # Build and package
```

---

## Code Style Guidelines

### TypeScript Conventions

**Interface Naming:**
- Entities: `PascalCase` (e.g., `Employee`, `Product`)
- Props: `ComponentNameProps` (e.g., `LoginFormProps`)
- Context: `ContextNameType` (e.g., `AuthContextType`)

**Type Rules:**
- **NEVER** use `any` - define proper interfaces
- Use `Pick<T, K>` for partial interfaces
- Use unions for controlled values: `type Status = 'active' | 'inactive'`
- Enable strict mode - `noUnusedLocals`, `noUnusedParameters` enforced

### Component Structure (Enforced Order)

```typescript
const ComponentName: React.FC<ComponentNameProps> = ({ prop1, prop2 }) => {
  // 1. Hooks (useState, useEffect, useContext)
  const [state, setState] = useState(initialValue);
  const { globalState } = useContext();

  // 2. Event handlers
  const handleClick = () => { /* ... */ };

  // 3. Computed values (useMemo)
  const computed = useMemo(() => expensiveCalc(state), [state]);

  // 4. Effects (useEffect)
  useEffect(() => { /* ... */ }, [dependencies]);

  // 5. Render
  return <div>...</div>;
};
```

### Import Order

1. React hooks (`React`, `useState`, `useEffect`)
2. External libraries (`lucide-react` icons)
3. Local contexts (`../../contexts/`)
4. Local components (`../components/`)
5. Types (`../../types/`)
6. Utils (`../../utils/`)

### File Organization

```
src/
├── components/
│   ├── Auth/           # Feature-specific folders
│   ├── Layout/
│   └── VirtualKeyboard.tsx  # Shared at root
├── contexts/           # React contexts
├── pages/             # Page components
├── types/             # TypeScript interfaces
├── services/          # API/service layers
├── hooks/             # Custom hooks
└── utils/             # Helper functions
```

### React Patterns

**State Management:**
- `useState`: Component-specific UI state
- `useReducer`: Complex state logic
- Context: Global app state (auth, cart)
- `useMemo`/`useCallback`: Performance optimization

**Component Patterns:**
- Use `React.memo` for components with frequent re-renders
- Memoize expensive calculations with `useMemo`
- Memoize handlers passed to children with `useCallback`

### Error Handling

```typescript
// Async error handling pattern
try {
  const result = await asyncOperation();
  setState({ data: result, loading: false, error: null });
} catch (error) {
  setState({ 
    data: null, 
    loading: false, 
    error: error instanceof Error ? error.message : 'Unknown error' 
  });
}

// UI Error display
{error && (
  <div className="bg-red-50 border-2 border-red-200 rounded-2xl p-6">
    <p className="text-red-700 text-xl font-medium">{error}</p>
  </div>
)}
```

### Tailwind CSS Guidelines

**Touch Screen Optimization (CRITICAL):**
- Minimum touch targets: `min-h-touch` (`3.75rem`, ~60px at 16px root); secondary `min-h-touch-sm`; compact `min-h-touch-xs` (~44px)
- Large action buttons: `min-h-20` (~5rem)
- Employee cards: `min-h-70` (~17.5rem tall target at default root)
- Avoid dropdowns and hover-dependent interactions

**Spacing Scale:**
- `gap-2` (8px): Tight spacing (keyboard keys)
- `gap-4` (16px): Standard spacing
- `gap-6` (24px): Comfortable spacing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portoyounes01/easysoft](https://github.com/portoyounes01/easysoft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
