---
trigger: always_on
description: **Security, Stability, and Best Practices - ALWAYS follow these principles:**
---

# Next.js Quick Starter - Cursor Rules

## 🎯 Critical Requirements

**Security, Stability, and Best Practices - ALWAYS follow these principles:**

### **1. Security-First Development**
- ✅ **Runtime Validation** - Use **Zod** to validate inputs at runtime (forms, API requests, environment variables, database queries)
- ✅ **Environment Security** - Never expose secrets client-side; use `NEXT_PUBLIC_` only for safe values
- ✅ **Authentication** - Implement **better-auth** for type-safe, session-based authentication
- ✅ **Authorization** - Always verify user permissions in server actions and API routes
- ✅ **Input Sanitization** - Validate and sanitize all user inputs before processing or storing
- ✅ **Database Security** - Use parameterized queries (Drizzle ORM handles this) to prevent SQL injection

### **2. Type Safety Everywhere**
- ✅ **TypeScript Strict Mode** - Enable strict type checking for compile-time safety
- ✅ **Zod for Runtime Types** - Validate data at runtime with Zod schemas
- ✅ **Type Inference** - Use `z.infer<>` for Zod types and `$inferSelect/$inferInsert` for Drizzle
- ✅ **No `any` Types** - Avoid `any`; use `unknown` or proper types
- ✅ **Explicit Return Types** - Define return types for all functions

### **3. Latest Stable Versions**
- ✅ **Node.js** - Use latest LTS version (currently Node.js 24 LTS)
- ✅ **Next.js** - Use latest stable (Next.js 16+ with App Router)
- ✅ **React** - Use latest stable (React 19+ with RSC and new hooks)
- ✅ **TypeScript** - Use latest stable (TypeScript 5+)
- ✅ **Package Manager** - Use **pnpm** (faster, more efficient than npm/yarn)

### **4. Architecture & Patterns**
- ✅ **Server Components First** - Default to React Server Components; use Client Components only when needed
- ✅ **Server Actions** - Use server actions for mutations with progressive enhancement
- ✅ **Separation of Concerns** - Keep business logic in server actions, UI in components
- ✅ **Error Boundaries** - Implement error.tsx for graceful error handling
- ✅ **Loading States** - Use loading.tsx and Suspense for better UX

### **5. Database & Data Integrity**
- ✅ **Drizzle ORM** - Use type-safe ORM with schema-first approach
- ✅ **drizzle-zod Integration** - Generate Zod schemas from Drizzle for single source of truth
- ✅ **Migrations** - Always use migrations (never push directly to production)
- ✅ **Data Validation** - Validate at database schema level AND application level
- ✅ **Transactions** - Use transactions for multi-step operations

### **6. Code Quality & Consistency**
- ✅ **Biome Linting** - Strict linting with consistent formatting (tabs, double quotes)
- ✅ **Code Comments** - Write descriptive JSDoc comments for functions and complex logic
- ✅ **Git Hooks** - Use Lefthook to enforce quality checks pre-commit
- ✅ **Descriptive Naming** - Use clear, self-documenting variable and function names
- ✅ **DRY Principle** - Don't repeat yourself; extract reusable logic

### **7. Testing & Reliability**
- ✅ **Test Critical Paths** - Write tests for authentication, data validation, and business logic
- ✅ **Type Check** - Run `tsc --noEmit` to catch type errors
- ✅ **Validate Schemas** - Test Zod schemas with valid and invalid inputs
- ✅ **Error Handling** - Always handle errors gracefully with try-catch and user-friendly messages

### **8. Performance & Optimization**
- ✅ **React Query** - Use TanStack React Query for efficient data fetching and caching
- ✅ **Code Splitting** - Use dynamic imports for large components
- ✅ **Image Optimization** - Use next/image for automatic optimization
- ✅ **Streaming** - Leverage Suspense and streaming for better perceived performance
- ✅ **Caching Strategy** - Implement proper caching with revalidation

### **9. Production Readiness**
- ✅ **Environment Validation** - Use `@next/env` + Zod to validate env vars at startup
- ✅ **Logging** - Implement proper error logging (console.error minimum, consider external service)
- ✅ **Monitoring** - Set up health checks and error tracking
- ✅ **Security Headers** - Configure proper CORS, CSP, and security headers
- ✅ **Rate Limiting** - Implement rate limiting for API routes in production

### **10. Developer Experience**
- ✅ **Documentation** - Maintain up-to-date README with setup instructions
- ✅ **.env.example** - Provide example environment file with all required variables
- ✅ **Local Development** - Support local SQLite database for easy development
- ✅ **Type-Safe Imports** - Use path aliases (@/) for cleaner imports
- ✅ **Clear Error Messages** - Provide helpful error messages with context

## Project Overview
This is a Next.js 16 application using React 19, Vercel AI SDK, Drizzle ORM, shadcn/ui, and Tailwind CSS v4.

## Core Technologies & Dependencies

### Framework & Runtime
- **Next.js 16+** - Latest stable with App Router and React Server Components
- **React 19+** - Latest stable with useActionState, useFormStatus, useOptimistic
- **TypeScript 5+** - Latest stable with strict mode enabled
- **Node.js 24 LTS** - Current LTS version (use latest LTS available)

### UI & Styling
- **shadcn/ui** - Component library with New York style and RSC support
- **Tailwind CSS v4** - Latest stable with CSS variables and modern features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abidjappie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
