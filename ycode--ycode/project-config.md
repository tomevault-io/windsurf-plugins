---
trigger: always_on
description: **YCode** is a self-hosted visual website builder built with Next.js 15, TypeScript, Tailwind CSS, and Supabase. This is a **fast-moving team project** where multiple developers use Cursor simultaneously. Code consistency and abstraction are critical.
---

# YCode Cursor AI Rules

## 🎯 Project Context

**YCode** is a self-hosted visual website builder built with Next.js 15, TypeScript, Tailwind CSS, and Supabase. This is a **fast-moving team project** where multiple developers use Cursor simultaneously. Code consistency and abstraction are critical.

---

## 🧠 Core Philosophy: ALWAYS Think About Abstraction

Before writing any code, ask yourself:
1. **Can this logic be extracted into a reusable hook?**
2. **Does a ShadCN component already exist for this UI element?**
3. **Is this component doing too many things? Should it be split?**
4. **Can this be a pure utility function instead of component logic?**
5. **Will another team member be able to reuse this easily?**

**Golden Rule**: If you're writing the same code twice, abstract it.

---

## 🎨 1. ShadCN Components (MANDATORY)

### ALWAYS Use ShadCN for UI Primitives

**Available ShadCN Components** (in `/components/ui/`):
- `Button` - All button interactions
- `Input` - Text inputs
- `Textarea` - Multi-line text
- `Label` - Form labels
- `Select` - Dropdowns
- `Tabs` - Tab navigation
- `Dropdown Menu` - Context menus
- `Popover` - Overlays
- `Tooltip` - Hover hints
- `Slider` - Range inputs
- `Badge` - Status indicators
- `Spinner` - Loading states
- `Icon` - Lucide icons (via icon.tsx)

### ✅ CORRECT Usage

```typescript
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Icon } from '@/components/ui/icon';

export function SearchBar({ onSearch }: SearchBarProps) {
  const [query, setQuery] = useState('');

  return (
    <div className="flex gap-2">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search pages..."
      />
      <Button size="sm" onClick={() => onSearch(query)}>
        <Icon name="search" />
        Search
      </Button>
    </div>
  );
}
```

### ❌ FORBIDDEN - Never Do This

```typescript
// ❌ WRONG: Custom button instead of ShadCN
export function SearchBar({ onSearch }: SearchBarProps) {
  return (
    <button className="px-4 py-2 bg-blue-500 rounded hover:bg-blue-600">
      Search
    </button>
  );
}

// ❌ WRONG: Custom input instead of ShadCN
<input className="border rounded px-2 py-1" />
```

### Extending ShadCN Components

If you need custom behavior, **compose** ShadCN components:

```typescript
// ✅ CORRECT: Composition
import { Button } from '@/components/ui/button';
import { Icon } from '@/components/ui/icon';

interface IconButtonProps {
  icon: string;
  label: string;
  onClick: () => void;
}

export function IconButton({ icon, label, onClick }: IconButtonProps) {
  return (
    <Button size="sm" variant="secondary" onClick={onClick}>
      <Icon name={icon} />
      {label}
    </Button>
  );
}
```

---

## 🏗️ 2. Component Architecture & Abstraction

### Single Responsibility Principle

Each component should do ONE thing well:

```typescript
// ✅ GOOD: Small, focused components
function PageListItem({ page, onSelect, isActive }: PageListItemProps) {
  return (
    <button
      onClick={() => onSelect(page.id)}
      className={cn(
        'w-full text-left px-3 py-2 rounded',
        isActive ? 'bg-zinc-700' : 'hover:bg-zinc-800'
      )}
    >
      <Icon name="file" />
      {page.title}
    </button>
  );
}

function PageList({ pages, selectedId, onSelect }: PageListProps) {
  return (
    <div className="flex flex-col gap-1">
      {pages.map(page => (
        <PageListItem
          key={page.id}
          page={page}
          isActive={selectedId === page.id}
          onSelect={onSelect}
        />
      ))}
    </div>
  );
}

// ❌ BAD: Monolithic component doing everything
function PageManagement() {
  // 500 lines of code managing pages, settings, modals, etc.
}
```

### Extract Custom Hooks

Move reusable logic into custom hooks:

```typescript
// ✅ GOOD: Reusable hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage in component
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      performSearch(debouncedQuery);
    }
  }, [debouncedQuery]);
}
```

### Extract Utility Functions

Keep components clean by moving pure functions to `/lib`:

```typescript
// ✅ GOOD: Utility in /lib/layer-utils.ts
export function findLayerById(layers: Layer[], id: string): Layer | null {
  for (const layer of layers) {
    if (layer.id === id) return layer;
    if (layer.children) {
      const found = findLayerById(layer.children, id);
      if (found) return found;
    }
  }
  return null;
}

// Usage in component
import { findLayerById } from '@/lib/layer-utils';

function LayerEditor({ layers, selectedId }: LayerEditorProps) {
  const selectedLayer = findLayerById(layers, selectedId);
  // ...
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ycode/ycode](https://github.com/ycode/ycode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
