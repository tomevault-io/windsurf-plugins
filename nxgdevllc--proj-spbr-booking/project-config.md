---
trigger: always_on
description: - **Always use PostgreSQL syntax** - this is a Supabase project
---

# 🎯 Cursor Rules for San Pedro Beach Resort Project

## 🗄️ Database & PostgreSQL/Supabase Rules

### Database Schema & Migration Rules

- **Always use PostgreSQL syntax** - this is a Supabase project
- **Create migrations in `docs/` folder** with descriptive names: `YYYY-MM-DD_description.sql`
- **Use snake_case** for all table names, column names, and function names
- **Always include `id UUID DEFAULT gen_random_uuid() PRIMARY KEY`** for new tables
- **Include `created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()`** and `updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()`** in all tables
- **Use appropriate data types**: 
  - `UUID` for IDs and foreign keys
  - `TEXT` for strings (not VARCHAR)
  - `NUMERIC(10,2)` for currency amounts
  - `BOOLEAN` for true/false values
  - `JSONB` for complex data structures
  - `TIMESTAMP WITH TIME ZONE` for dates

### Row Level Security (RLS) Rules
- **Enable RLS on ALL tables** with `ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;`
- **Create policies for each operation** (SELECT, INSERT, UPDATE, DELETE)
- **Use role-based access control** with these roles: 'admin', 'manager', 'employee', 'guest'
- **Always check user authentication** with `auth.uid()` in policies
- **Create policies in this order**:
  1. SELECT policies (read access)
  2. INSERT policies (create access)
  3. UPDATE policies (modify access)
  4. DELETE policies (remove access)

### Supabase-Specific Rules
- **Use Supabase client** from `@/lib/supabase` for all database operations
- **Never expose service role key** in client-side code
- **Use environment variables** for all Supabase configuration
- **Implement proper error handling** for all Supabase operations
- **Use real-time subscriptions** only when necessary
- **Enable proper CORS settings** in Supabase dashboard

### Database Naming Conventions
- **Tables**: `snake_case` (e.g., `user_profiles`, `rental_units_pricing`)
- **Columns**: `snake_case` (e.g., `first_name`, `check_in_date`)
- **Functions**: `snake_case` (e.g., `get_user_role`, `calculate_total`)
- **Policies**: Descriptive names (e.g., "Users can view own profile")
- **Indexes**: `idx_table_column` (e.g., `idx_user_profiles_role`)

## 🎨 Frontend & React/Next.js Rules

### Component Structure
- **Use TypeScript** for all components and functions
- **Create components in `src/components/`** with PascalCase naming
- **Use functional components** with hooks, not class components
- **Implement proper prop types** with TypeScript interfaces
- **Use `'use client'`** directive for client-side components
- **Keep components small and focused** on single responsibility

### Styling Rules
- **Use Tailwind CSS** for all styling
- **Follow the project's color scheme**: green and yellow theme
- **Use semantic class names** and avoid custom CSS when possible
- **Implement responsive design** with mobile-first approach
- **Use consistent spacing** with Tailwind's spacing scale
- **Apply dark theme** with proper contrast ratios

### State Management
- **Use React hooks** (useState, useEffect, useContext) for state
- **Implement proper loading states** for all async operations
- **Handle errors gracefully** with user-friendly messages
- **Use optimistic updates** where appropriate
- **Implement proper form validation** with error states

## 🔐 Security Rules

### Authentication & Authorization
- **Always check user permissions** before sensitive operations
- **Implement role-based access control** in both frontend and backend
- **Use secure session management** with proper token handling
- **Validate all user inputs** on both client and server side
- **Implement proper logout functionality** with token cleanup

### Data Protection
- **Never store sensitive data** in localStorage or sessionStorage
- **Use environment variables** for all API keys and secrets
- **Implement proper data sanitization** for all user inputs
- **Use HTTPS** for all external communications
- **Implement rate limiting** for API endpoints

## 📁 File Organization Rules

### Project Structure
```
src/
├── app/                 # Next.js app router pages
├── components/          # Reusable React components
├── lib/                 # Utility functions and configurations
├── types/               # TypeScript type definitions
└── hooks/               # Custom React hooks
```

### File Naming Conventions
- **Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- **Pages**: `page.tsx` (Next.js app router)
- **Utilities**: `camelCase.ts` (e.g., `formatCurrency.ts`)
- **Types**: `camelCase.ts` (e.g., `userTypes.ts`)
- **Constants**: `UPPER_SNAKE_CASE.ts` (e.g., `API_ENDPOINTS.ts`)

## 🧪 Testing & Quality Rules

### Code Quality
- **Use ESLint** for code linting and formatting
- **Follow TypeScript strict mode** guidelines
- **Implement proper error boundaries** for React components
- **Use meaningful variable and function names**
- **Add JSDoc comments** for complex functions
- **Keep functions small** (max 20-30 lines)

### Performance Rules
- **Use React.memo()** for expensive components
- **Implement proper loading states** and skeleton screens
- **Use Next.js Image component** for optimized images
- **Implement proper caching strategies** for API calls
- **Use code splitting** for large components

## 📊 Data Import & CSV Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nxgdevllc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
