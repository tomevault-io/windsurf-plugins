---
trigger: always_on
description: Comprehensive coding standards and best practices for the AltMount React + TypeScript frontend.
---

# AltMount Frontend Development Standards

Comprehensive coding standards and best practices for the AltMount React + TypeScript frontend.

## React Best Practices

### Component Structure

```tsx
// ✅ Good: Functional component with TypeScript
interface ComponentProps {
  title: string;
  isActive?: boolean;
  onAction: (id: string) => void;
}

export function ComponentName({ title, isActive = false, onAction }: ComponentProps) {
  const [state, setState] = useState<string>("");
  
  return (
    <div className="component-container">
      {/* Component content */}
    </div>
  );
}

// ❌ Avoid: Default exports, arrow functions for components
export default ({ title }) => { /* ... */ };
```

### TypeScript Guidelines

- **Always define interfaces** for component props and complex objects
- **Use strict typing** - avoid `any` type
- **Define return types** for complex functions
- **Use union types** for state enums and options
- **Run type checking**: Use `bun run check` for comprehensive TypeScript validation

```tsx
// ✅ Good: Strict typing
interface UserStatus {
  status: 'online' | 'offline' | 'away';
  lastSeen?: Date;
}

// ❌ Avoid: Loose typing
interface UserStatus {
  status: string;
  lastSeen: any;
}
```

### State Management

- **Use `useState` for local component state**
- **Use custom hooks** for shared state logic
- **Keep state minimal** - derive computed values
- **Batch state updates** when possible

```tsx
// ✅ Good: Custom hook for API state
function useConfig() {
  const [data, setData] = useState<ConfigResponse | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  // Hook logic here
  return { data, isLoading, error, refetch };
}

// ✅ Good: Using the hook
function ConfigPage() {
  const { data: config, isLoading, error, refetch } = useConfig();
  // Component logic
}
```

### Hook Best Practices

- **Custom hooks start with `use`**
- **Return objects not arrays** for multiple values
- **Use `useCallback` for event handlers** passed to children
- **Use `useMemo` for expensive calculations**

```tsx
// ✅ Good: Custom hook with clear return
function useApi<T>(endpoint: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    try {
      const response = await fetch(endpoint);
      const result = await response.json();
      setData(result);
    } finally {
      setLoading(false);
    }
  }, [endpoint]);
  
  return { data, loading, fetchData };
}
```

## DaisyUI Component Guidelines

### Prefer DaisyUI Components

Always use DaisyUI components over custom CSS when available:

```tsx
// ✅ Good: Use DaisyUI components
<button type="button" className="btn btn-primary">
  Primary Action
</button>

<div className="card bg-base-100 shadow-lg">
  <div className="card-body">
    <h2 className="card-title">Card Title</h2>
    <p>Card content here</p>
  </div>
</div>

// ❌ Avoid: Custom styling when DaisyUI exists
<button 
  type="button" 
  className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
>
  Custom Button
</button>
```

### DaisyUI Component Patterns

#### Buttons

```tsx
// Basic buttons
<button type="button" className="btn">Default</button>
<button type="button" className="btn btn-primary">Primary</button>
<button type="button" className="btn btn-secondary">Secondary</button>
<button type="button" className="btn btn-outline">Outline</button>

// Button sizes
<button type="button" className="btn btn-xs">Extra Small</button>
<button type="button" className="btn btn-sm">Small</button>
<button type="button" className="btn btn-lg">Large</button>

// Button states
<button type="button" className="btn btn-primary" disabled>Disabled</button>
<button type="button" className="btn btn-primary loading">Loading</button>
```

#### Cards

```tsx
<div className="card bg-base-100 shadow-lg">
  <div className="card-body">
    <h2 className="card-title">
      Card Title
      <div className="badge badge-secondary">NEW</div>
    </h2>
    <p>Card description text here</p>
    <div className="card-actions justify-end">
      <button type="button" className="btn btn-primary">Action</button>
    </div>
  </div>
</div>
```

#### Menus

```tsx
<ul className="menu bg-base-200 rounded-box">
  <li>
    <a className={activeItem === 'home' ? 'active' : ''}>
      <HomeIcon className="h-5 w-5" />
      Home
    </a>
  </li>
  <li>
    <a>
      <SettingsIcon className="h-5 w-5" />
      Settings
      <span className="badge badge-warning badge-xs">New</span>
    </a>
  </li>
</ul>
```

#### Forms

```tsx
// ✅ Good: Use DaisyUI fieldset for form inputs
<fieldset className="fieldset">
  <legend className="fieldset-legend">Input Label</legend>
  <input 
    type="text" 
    className="input" 
    placeholder="Enter text here"
  />
  <p className="label">Helper text</p>
</fieldset>

// ✅ Good: Fieldset with select dropdown
<fieldset className="fieldset">
  <legend className="fieldset-legend">Select Option</legend>
  <select className="select">
    <option value="">Choose an option</option>
    <option value="option1">Option 1</option>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javi11/altmount](https://github.com/javi11/altmount) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
