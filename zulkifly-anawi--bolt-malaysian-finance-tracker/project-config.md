---
trigger: always_on
description: > Repository-wide instructions for AI assistants working on this project.
---

# GitHub Copilot Instructions - Malaysian Financial Tracker

> Repository-wide instructions for AI assistants working on this project.
> This file provides context, conventions, and guidelines for generating accurate, consistent code.

## Project Overview

**Malaysian Financial Tracker** is a comprehensive financial tracking platform built specifically for Malaysians. It helps users manage investments (ASB, EPF, Tabung Haji), savings, and financial goals with beautiful visualizations and smart insights.

### Business Context
- **Target Users**: Malaysian individuals managing personal finances
- **Core Domain**: Financial tracking for Malaysian investment products
- **Key Products Supported**:
  - ASB (Amanah Saham Bumiputera) - Unit trust with annual dividends
  - EPF (Employees Provident Fund) - Retirement savings with Account 1/Account 2 split
  - Tabung Haji - Hajj savings with hibah (Islamic dividend)
  - Savings accounts, fixed deposits, unit trusts, stocks, crypto

### Privacy Principles
- **Manual Tracking Only**: We never connect to actual bank accounts
- **Zero Credentials**: We never ask for bank passwords
- **User Data Ownership**: Users can export all their data anytime
- **Supabase Only**: Financial data never leaves the Supabase infrastructure

---

## Tech Stack

### Frontend
- **Framework**: React 18.3.1 with TypeScript 5.5.3
- **Build Tool**: Vite 5.4.2
- **Styling**: Tailwind CSS with custom glassmorphism design
- **Icons**: Lucide React (only use icons from this library)
- **State Management**: React Context API (AuthContext)

### Backend
- **Database**: PostgreSQL via Supabase
- **Authentication**: Supabase Auth (Email/Password only)
- **Security**: Row Level Security (RLS) policies on all tables
- **Real-time**: Supabase real-time subscriptions (where applicable)

### Development
- **Package Manager**: npm
- **Linting**: ESLint with TypeScript rules
- **Type Checking**: `npm run typecheck`
- **Dev Server**: `npm run dev` (port 5173)

---

## Architecture Patterns

### Component Organization
```
src/components/
├── accounts/        # Account CRUD and display
├── admin/           # Admin panel (restricted to is_admin users)
│   └── pages/       # Admin sub-pages
├── common/          # Shared UI components (buttons, cards, modals)
├── engagement/      # Achievements, notifications, insights, export
├── goals/           # Goal CRUD and progress tracking
├── help/            # FAQ, onboarding, help system
├── investments/     # ASB, EPF, Tabung Haji calculators
└── [Component].tsx  # Top-level shared components
```

### State Management
- **AuthContext**: Global authentication state, user profile, `isAdmin` flag
- **Local State**: Component-level state with `useState`, `useEffect`
- **No Redux**: Keep state management simple with Context + hooks

### Data Flow
1. **Supabase Client** (`src/lib/supabase.ts`) - Single instance
2. **Services** (`src/services/`) - Business logic and API calls
3. **Components** - UI rendering and user interaction
4. **Types** (`src/types/database.ts`) - Shared TypeScript interfaces

---

## Code Conventions

### TypeScript
```typescript
// ✅ DO: Use explicit types
interface AccountProps {
  account: Account;
  onUpdate: (account: Account) => void;
}

// ❌ DON'T: Use 'any' type
function process(data: any) { ... }

// ✅ DO: Use type imports
import type { Profile, Account, Goal } from '../types/database';

// ✅ DO: Define interfaces for component props
export const AccountCard = ({ account, onUpdate }: AccountProps) => { ... }
```

### React Components
```typescript
// ✅ DO: Use functional components with TypeScript
export const MyComponent = ({ prop1, prop2 }: MyComponentProps) => {
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    // Effect logic
  }, [dependencies]);
  
  return (
    <div className="...">
      {/* JSX */}
    </div>
  );
};

// ❌ DON'T: Use class components
class MyComponent extends React.Component { ... }
```

### Naming Conventions
- **Components**: PascalCase (`AccountCard.tsx`, `GoalProgress.tsx`)
- **Hooks**: camelCase with `use` prefix (`useConfig.ts`)
- **Services**: camelCase with `Service` suffix (`adminConfigService.ts`)
- **Utils**: camelCase (`formatters.ts`, `validation.ts`)
- **Types**: PascalCase for interfaces (`Profile`, `Account`, `Goal`)

### File Structure
```typescript
// Component file structure
import { useState, useEffect } from 'react';
import { IconName } from 'lucide-react';
import { supabase } from '../lib/supabase';
import type { TypeName } from '../types/database';

interface ComponentProps {
  // Props definition
}

export const ComponentName = ({ props }: ComponentProps) => {
  // State declarations
  // Effects
  // Event handlers
  // Return JSX
};
```

---

## Styling Guidelines

### Tailwind CSS
```jsx
// ✅ DO: Use Tailwind utility classes
<div className="bg-white/10 backdrop-blur-sm rounded-2xl p-6 border border-white/20">

// ✅ DO: Use glassmorphism pattern (project standard)
<div className="glass-card"> {/* Custom class in index.css */}

// ❌ DON'T: Use inline styles
<div style={{ backgroundColor: 'rgba(255,255,255,0.1)' }}>

// ❌ DON'T: Create new CSS files
```

### Color Palette
- **Background**: Dark theme with glass-morphic overlays

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zulkifly-anawi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
