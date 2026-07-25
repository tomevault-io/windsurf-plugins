---
trigger: always_on
description: This is a monorepo containing a React-based admin console, user portal and features as individual NPM packages for the WSO2 Identity Server. It uses Nx for monorepo build management, pnpm for package management, and changeset for version management.
---

## Project Overview

This is a monorepo containing a React-based admin console, user portal and features as individual NPM packages for the WSO2 Identity Server. It uses Nx for monorepo build management, pnpm for package management, and changeset for version management.

## Essential Commands

### Build

```bash
pnpm install && pnpm build        # Full build (all apps + modules)
pnpm build:modules                # Build shared modules only
pnpm build:apps                   # Build console and myaccount apps
```

### Development

```bash
cd apps/console && pnpm start     # Console at https://localhost:9001/console
cd apps/myaccount && pnpm start   # My Account at https://localhost:9000/myaccount
```

### Testing

```bash
pnpm test                         # Run all unit tests
pnpm test:watch                   # Watch mode

# Single test file (from features directory)
npx vitest run features/admin.applications.v1/__tests__/applications-page.test.tsx

# Single module tests
cd modules/core && pnpm test
```

### Linting & Type Checking

```bash
pnpm lint                         # ESLint all projects
pnpm lint:autofix                 # Auto-fix issues
pnpm typecheck                    # TypeScript type checking
```

### Cleanup

```bash
pnpm clean                        # Clean all build artifacts
```

## Architecture

### Directory Structure

- **apps/console** - Admin console (port 9001)
- **apps/myaccount** - User self-service portal (port 9000)
- **modules/** - Shared libraries (core, react-components, theme, i18n, forms, validation, etc.)
- **features/** - Feature modules (70+ admin.*.v1 packages for console features)
- **identity-apps-core/** - JSP portals (auth portal, recovery portal)

### Key Modules

- `@wso2is/core` - API utilities, configs, hooks, Redux store, helpers
- `@wso2is/react-components` - Semantic UI-based component library
- `@wso2is/theme` - Theming system
- `@wso2is/i18n` - Internationalization (i18next)
- `@wso2is/forms` - Unified form library (Oxygen UI / MUI field adapters) in `modules/forms`

### Technology Stack

- React 18, TypeScript 5.1, Redux
- Vite, Nx 21, pnpm 10
- Vitest + React Testing Library
- Semantic UI React + MUI + Oxygen UI (WSO2 design system)

## Code Conventions

### Naming Conventions

- **Interfaces**: Must use `Interface` suffix (e.g., `ApplicationListInterface`, `FeatureConfigInterface`)
- **Component props**: Define prop interfaces in the component file, extend `IdentifiableComponentInterface`. Only move to `models/` if shared across multiple components
- **Component IDs**: Use hierarchical naming: `${parentId}-${childName}` (e.g., `wizard-color-picker`)

### TypeScript Conventions

This codebase uses **explicit type annotations everywhere** — do not rely on type inference. Never use `any`; use proper types or `unknown` with type guards.

**Variables** — always annotate, even when the type is obvious:

```typescript
const isLoading: boolean = true;
const name: string = config.appName;
const items: ApplicationListInterface[] = [];
const dispatch: Dispatch = useDispatch();
```

**Components** — use `FunctionComponent` (never `FC` or `React.FC`) with `ReactElement` return:
```typescript
const MyComponent: FunctionComponent<MyComponentPropsInterface> = (
    props: MyComponentPropsInterface
): ReactElement => { ... };
```

**Hooks** — explicit generics on `useState`, `useRef`; explicit type on `useSelector`:
```typescript
const [ isOpen, setIsOpen ] = useState<boolean>(false);
const timerRef: React.MutableRefObject<number | null> = useRef<number | null>(null);
const featureConfig: FeatureConfigInterface = useSelector(
    (state: AppState) => state?.config?.ui?.features
);
```

**Callbacks** — type the variable with the full function signature AND the arrow function return type:
```typescript
const handleSubmit: (name: string) => void = useCallback(
    (name: string): void => { ... },
    []
);
```

**useMemo** — type the variable with the expected result type:
```typescript
const isValid: boolean = useMemo(() => name.length > 0, [ name ]);
```

**Async functions** — annotate with `Promise<T>` on both variable and function:
```typescript
const createApp: () => Promise<void> = useCallback(async (): Promise<void> => {
    const result: CreatedAppResultInterface = await createOnboardingApplication(data);
}, [ data ]);
```

**Custom hook return types** — define a named return interface:
```typescript
interface UseMyHookReturn {
    isLoading: boolean;
    data: MyDataInterface | null;
}

export const useMyHook = (): UseMyHookReturn => { ... };
```

### Testing

- Test files: `__tests__/<component>.test.tsx` adjacent to component
- Use `data-componentid` for DOM element selection (not id/classes)
- Use custom render from `@wso2is/unit-testing/utils` (wraps with providers)
- Mock APIs with MSW (handlers in `test-configs/__mocks__/server`)
- `data-testid` is deprecated - use `data-componentid` via `IdentifiableComponentInterface`

### Components

- Extend `IdentifiableComponentInterface` from `@wso2is/core/models` for component IDs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wso2/identity-apps](https://github.com/wso2/identity-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
