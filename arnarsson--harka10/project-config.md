---
trigger: always_on
description: - **ALWAYS test authentication logic** when making auth/routing changes
---

# Claude Code Instructions for HARKA Project

## Testing Strategy (Updated for Production Environment)

### Core Testing Principles
- **ALWAYS test authentication logic** when making auth/routing changes
- **VALIDATE middleware behavior** through code review and manual testing
- **TEST build compilation** after every change: `pnpm run build`
- **VERIFY TypeScript compilation** with no errors
- **MANUAL verification** of key user flows on deployed app

### Required Testing Scenarios
When making auth/routing changes, manually verify these scenarios:

1. **Unauthenticated Access**:
   - Dashboard URL → Should redirect to sign-in
   - Protected routes → Should redirect to sign-in
   - Public routes (/, /toolkit) → Should load normally

2. **Authentication Flow**:
   - Sign-in page loads without errors
   - Sign-up page loads without errors
   - No redirect loops on auth pages
   - Successful auth redirects to dashboard

3. **Navigation**:
   - Homepage login/signup buttons work
   - Dashboard link redirects unauthenticated users
   - Authenticated users can access protected routes

### Middleware Testing Checklist
When modifying `middleware.ts`, verify:

```typescript
// Key middleware logic patterns to maintain:

1. Auth page check FIRST (prevents redirect loops)
   if (userId && isAuthPage(req)) {
     return NextResponse.redirect(new URL('/learn/dashboard', req.url))
   }

2. Admin route protection
   if (isAdminRoute(req) && !req.nextUrl.pathname.startsWith('/admin/sign-in')) {
     // Admin logic here
   }

3. Public route allowance
   if (isPublicRoute(req)) {
     return NextResponse.next()
   }

4. Protected route authentication requirement
   if (!userId) {
     return NextResponse.redirect(new URL('/sign-in', req.url))
   }
```

## Code Quality Standards

### Authentication & Security
- **NEVER** expose API keys or secrets in client-side code
- **ALWAYS** validate user authentication on server-side (middleware)
- **USE** proper TypeScript types for auth state
- **IMPLEMENT** proper error boundaries for auth failures
- **AVOID** conflicting redirect URLs in components and providers

### Clerk Configuration Rules
- **MINIMAL** ClerkProvider configuration (avoid inline URLs)
- **USE** environment variables for redirect URLs when needed
- **AVOID** multiple redirect URL configs (component + provider + env)
- **REMOVE** `forceRedirectUrl` and `fallbackRedirectUrl` from components
- **LET** middleware handle routing logic

### Performance Optimization
- **USE** Next.js Image component for all images
- **IMPLEMENT** proper loading states and skeleton screens
- **MINIMIZE** client-side JavaScript bundles
- **USE** dynamic imports for heavy components
- **OPTIMIZE** database queries and API calls
- **IMPLEMENT** proper caching strategies

### Code Structure
- **FOLLOW** existing patterns and conventions
- **USE** TypeScript for all new code
- **IMPLEMENT** proper error handling
- **WRITE** self-documenting code with clear variable names
- **AVOID** deeply nested components (max 3 levels)
- **USE** composition over inheritance

### Component Standards
- **CREATE** reusable components in `/components`
- **USE** proper prop types and interfaces
- **IMPLEMENT** loading and error states
- **ENSURE** accessibility (ARIA labels, keyboard navigation)
- **AVOID** inline styles, use Tailwind classes

### API & Data Handling
- **USE** proper HTTP status codes
- **IMPLEMENT** request/response validation
- **HANDLE** network errors gracefully
- **USE** proper data fetching patterns (SWR, React Query, etc.)
- **VALIDATE** all user inputs

## Development Workflow

### Before Making Changes
1. **READ** existing code to understand patterns
2. **CHECK** if similar functionality exists
3. **PLAN** the implementation approach
4. **IDENTIFY** potential breaking changes

### During Development
1. **FOLLOW** TypeScript best practices
2. **IMPLEMENT** proper error handling
3. **ADD** loading states for async operations
4. **MAINTAIN** consistent code style
5. **AVOID** introducing new dependencies without justification

### After Changes
1. **RUN** build: `pnpm run build`
2. **CHECK** TypeScript compilation succeeds
3. **VERIFY** no console errors in development
4. **TEST** key user flows manually
5. **DEPLOY** and verify on production URL

### Build Verification Commands
```bash
# Essential build checks
pnpm run build          # Must pass without errors
pnpm run dev            # Should start without issues
pnpm run type-check     # TypeScript compilation check

# Development workflow
pnpm run dev            # Start development server
pnpm run lint           # Check code quality
```

## Project-Specific Rules

### Authentication (Clerk)
- **USE** middleware for route protection
- **AVOID** client-side route guards
- **IMPLEMENT** proper loading states during auth checks
- **KEEP** ClerkProvider configuration minimal
- **USE** environment variables for redirect configuration

### Middleware Configuration
```typescript
// Correct pattern for middleware.ts:
const isPublicRoute = createRouteMatcher([
  "/", "/about", "/pricing", "/contact", "/blog", "/blog/(.*)",
  "/sign-in", "/sign-in/(.*)", "/sign-up", "/sign-up/(.*)",
  "/login", "/signup", "/logout", "/toolkit", "/team", "/workshop"
])


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Arnarsson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
