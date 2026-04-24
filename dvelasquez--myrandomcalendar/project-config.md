---
trigger: always_on
description: - React components should be **Pure**: Data flows in one direction
---

# React-Specific Rules

## 🎯 React Architecture Principles

### **Pure Functional Components**
- React components should be **Pure**: Data flows in one direction
- Only manage presentational state, **NO DATA FETCHING**
- If something in React needs to make a POST update, it happens on a form
- Use props for all data, avoid `useEffect` for initial data loading

### **Component Design**
- Prefer functional components over class components
- Use TypeScript for all component props and state
- Keep components small and focused on single responsibility
- Use composition over inheritance

## 🚫 What NOT to Do in React Components

### **Avoid Data Fetching**
```typescript
// ❌ DON'T DO THIS
const [data, setData] = useState([]);

useEffect(() => {
  // Don't fetch initial data in React
  fetchData().then(setData);
}, []);
```

### **Avoid Complex State Management**
```typescript
// ❌ DON'T DO THIS
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setLoading(true);
  fetchUser().then(setUser).catch(setError).finally(() => setLoading(false));
}, []);
```

### **Avoid Custom Button Styling**
```typescript
// ❌ DON'T DO THIS
<button className="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Click me
</button>

<a className="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">
  Link button
</a>
```

## ✅ What TO Do in React Components

### **Accept Data as Props**
```typescript
// ✅ DO THIS
interface DashboardProps {
  user: User;
  data: DashboardData[];
  onUpdate: (id: string, data: Partial<DashboardData>) => void;
}

export function Dashboard({ user, data, onUpdate }: DashboardProps) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      {data.map(item => (
        <DashboardItem key={item.id} item={item} onUpdate={onUpdate} />
      ))}
    </div>
  );
}
```

### **Handle Form Submissions**
```typescript
// ✅ DO THIS
interface FormProps {
  onSubmit: (data: FormData) => void;
  initialData?: Partial<FormData>;
}

export function UserForm({ onSubmit, initialData }: FormProps) {
  const [formData, setFormData] = useState(initialData || {});

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
    </form>
  );
}
```

## 🎨 ShadCN Component Guidelines

### **Button Components (CRITICAL)**
- **Use `Button`** for all `<button>` elements
- **Use `ButtonLink`** for all `<a>` tags styled as buttons
- **Never use custom button styling** - only ShadCN variants
- **Import both components** when needed in Astro files

```typescript
// ✅ CORRECT: React components
import { Button } from './ui/button';

<Button type="submit" variant="destructive">
  Delete
</Button>

<Button variant="outline" size="sm">
  Edit
</Button>
```

```astro
<!-- ✅ CORRECT: Astro files -->
---
import { Button } from '@/components/ui/button';
import ButtonLink from '@/components/ui/button-link';
---

<Button type="submit" className="w-full">
  Submit
</Button>

<ButtonLink variant="secondary" href="/back">
  Back
</ButtonLink>
```

### **Variant Selection Rules**
- `default` - Primary actions (create, submit, main CTAs)
- `secondary` - Secondary actions (back, cancel, navigation)
- `destructive` - Delete, remove, dangerous actions
- `outline` - Alternative actions, form cancel buttons
- `ghost` - Subtle actions, close buttons, icon buttons
- `link` - Text links within content

```typescript
// ✅ CORRECT: Semantic variant usage
<Button variant="default">Create New Event</Button>        // Primary action
<Button variant="secondary">Cancel</Button>               // Secondary action
<Button variant="destructive">Delete</Button>             // Dangerous action
<Button variant="outline">Back to Schedule</Button>       // Alternative action
<Button variant="ghost" size="icon">×</Button>            // Close button
<ButtonLink variant="link" href="/login">Sign in</ButtonLink> // Text link
```

### **Size Guidelines**
- `default` - Standard buttons (most common)
- `sm` - Small buttons (edit, delete actions in lists)
- `lg` - Large buttons (hero CTAs, important actions)
- `icon` - Icon-only buttons (close, menu, etc.)

```typescript
// ✅ CORRECT: Size usage
<Button size="default">Primary Action</Button>
<Button size="sm" variant="destructive">Delete</Button>
<Button size="lg">Get Started</Button>
<Button size="icon" variant="ghost">×</Button>
```

## 🎨 Styling Guidelines

### **Use ShadCN Components First**
- **Prefer ShadCN components** over custom styling
- **Use TailwindCSS utilities** for layout and spacing
- **Use `className` prop** for React components
- **Use `class` prop** for Astro/HTML elements
- **Never override ShadCN component styles** - use variants instead

```typescript
// ✅ DO THIS: ShadCN components
<div className="flex items-center justify-between p-4">
  <h2 className="text-xl font-semibold">Title</h2>
  <Button variant="outline">Action</Button>
</div>

// ❌ DON'T DO THIS: Custom button styling
<button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
  Action
</button>
```

### **Component Composition**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvelasquez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
