---
trigger: always_on
description: Design system mastery: design tokens, shadcn/ui, Tailwind CSS v4, component composition, theming, animations, and icon systems.
---


# Design System Mastery

Comprehensive guide for building and using design systems in modern web applications.

## Design Tokens

### Color Tokens

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        // Semantic colors
        brand: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
          950: '#172554',
        },
        // Status colors
        success: {
          DEFAULT: '#22c55e',
          light: '#4ade80',
          dark: '#16a34a',
        },
        warning: {
          DEFAULT: '#eab308',
          light: '#facc15',
          dark: '#ca8a04',
        },
        error: {
          DEFAULT: '#ef4444',
          light: '#f87171',
          dark: '#dc2626',
        },
        // Surface colors
        surface: {
          primary: '#ffffff',
          secondary: '#f8fafc',
          tertiary: '#f1f5f9',
          muted: '#e2e8f0',
        },
      },
    },
  },
};
```

### Typography Tokens

```javascript
// tailwind.config.js
theme: {
  extend: {
    fontFamily: {
      sans: ['Inter var', 'system-ui', 'sans-serif'],
      display: ['Cal Sans', 'system-ui', 'sans-serif'],
      mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
    },
    fontSize: {
      'xs': ['0.75rem', { lineHeight: '1rem' }],
      'sm': ['0.875rem', { lineHeight: '1.25rem' }],
      'base': ['1rem', { lineHeight: '1.5rem' }],
      'lg': ['1.125rem', { lineHeight: '1.75rem' }],
      'xl': ['1.25rem', { lineHeight: '1.75rem' }],
      '2xl': ['1.5rem', { lineHeight: '2rem' }],
      '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
      '4xl': ['2.25rem', { lineHeight: '2.5rem' }],
    },
    fontWeight: {
      normal: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
    },
  },
}
```

### Spacing Tokens

```javascript
theme: {
  extend: {
    spacing: {
      '0': '0',
      '1': '0.25rem',
      '2': '0.5rem',
      '3': '0.75rem',
      '4': '1rem',
      '5': '1.25rem',
      '6': '1.5rem',
      '8': '2rem',
      '10': '2.5rem',
      '12': '3rem',
      '16': '4rem',
      '20': '5rem',
      '24': '6rem',
      // Component-specific
      'card-padding': '1.5rem',
      'section-gap': '4rem',
      'input-height': '2.5rem',
    },
  },
}
```

### Border Radius Tokens

```javascript
theme: {
  extend: {
    borderRadius: {
      none: '0',
      sm: '0.25rem',
      DEFAULT: '0.375rem',
      md: '0.5rem',
      lg: '0.75rem',
      xl: '1rem',
      '2xl': '1.5rem',
      '3xl': '2rem',
      full: '9999px',
      // Semantic
      card: '12px',
      button: '8px',
      input: '6px',
      avatar: '50%',
    },
  },
}
```

---

## shadcn/ui Patterns

### Initialization

```bash
# Initialize shadcn-ui
npx shadcn@latest init

# Add components
npx shadcn@latest add button card input label form
npx shadcn@latest add dialog dropdown-menu select
npx shadcn@latest add toast alert accordion
```

### Component Composition

```tsx
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Checkbox } from '@/components/ui/checkbox';

// Login form composition
function LoginForm() {
  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Welcome back</CardTitle>
        <CardDescription>
          Enter your email and password to sign in to your account.
        </CardDescription>
      </CardHeader>
      <CardContent>
        <form className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>
            <Input 
              id="email" 
              type="email" 
              placeholder="m@example.com"
              required 
            />
          </div>
          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>
            <Input 
              id="password" 
              type="password" 
              required 
            />
          </div>
          <div className="flex items-center space-x-2">
            <Checkbox id="remember" />
            <label
              htmlFor="remember"
              className="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
            >
              Remember me
            </label>
          </div>
        </form>
      </CardContent>
      <CardFooter className="flex flex-col space-y-4">
        <Button className="w-full">Sign in</Button>
        <p className="text-sm text-muted-foreground text-center">
          Don't have an account?{' '}
          <a href="/register" className="text-primary hover:underline">
            Sign up
          </a>
        </p>
      </CardFooter>
    </Card>
  );
}
```

### Dialog Patterns

```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
