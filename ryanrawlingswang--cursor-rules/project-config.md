---
trigger: always_on
description: NextJs development rules
---


# Next.js Development Rules

You are an expert full-stack TypeScript engineer working with Next.js. Produce production-ready code that follows modern Next.js App Router patterns and best practices.

## Tech Stack Guidelines

- **Next.js 15+** (App Router) + React 19
- **TypeScript** (strict mode)
- **Testing**: Vitest + React Testing Library + MSW
- **Styling**: Tailwind CSS + shadcn/ui (when applicable)
- **Package Manager**: pnpm

Focus on: separation of concerns, strict typing, accessibility, performance, and comprehensive testing.

## Project Structure

```
src/
├─ app/                       # App Router (RSC by default)
│  ├─ _components/            # App-level (RSC-ok) components
│  ├─ api/                    # API routes for external webhooks only
│  ├─ (routes)/               # Route groups and pages
│  └─ layout.tsx
├─ components/                # Reusable client/RSC components
│  ├─ ui/                     # Base UI components
│  └─ [feature]/              # Feature-specific components
├─ lib/                       # Utilities and shared helpers
├─ server/                    # Server-side code
│  ├─ api/                    # tRPC API layer
│  │  ├─ routers/             # tRPC router composition
│  │  ├─ procedures/          # Individual tRPC procedures
│  │  ├─ root.ts              # Root router
│  │  └─ trpc.ts              # tRPC setup
│  ├─ services/               # Business logic services
│  └─ clients/                # External service clients
├─ db/                        # Database schema & migrations
├─ tools/                     # AI tool definitions
├─ prompts/                   # AI prompt templates
└─ trpc/                      # tRPC client setup

test/
├─ unit/                      # Unit tests
├─ integ/                     # Integration tests
└─ e2e/                       # End-to-end tests
```

### Directory Conventions

- Use **kebab-case** for folders and files
- Co-locate tests next to source: `*.test.ts(x)`
- Export from `index.ts` barrels only when it helps DX without circular deps

## Next.js App Router Rules

### Server vs Client Components

- **Default to RSC** (no "use client"). Use client components only for:
  - Interactivity (event handlers)
  - Browser APIs (localStorage, window, etc.)
  - Third-party libraries requiring client-side initialization

### Server Boundaries

- Keep data access in server context (RSC, server actions, API routes)
- API routes in `app/api` are for external webhooks only
- Use internal APIs (tRPC, server actions) for app-to-app communication

### Performance & UX

- Use `<Suspense>` for async UI; provide skeletons
- Implement proper loading and error boundaries
- Use Next.js caching (`revalidateTag`, `revalidatePath`) appropriately
- Prefer Edge runtime for simple operations, Node for complex ones

### Accessibility

- All interactive elements must have proper roles and labels
- Ensure keyboard navigation support
- Provide ARIA attributes where needed
- Test with screen readers

## API Architecture

### Internal APIs (tRPC Recommended)

- Organize by feature domains
- Use Zod for input validation
- Implement proper error handling with custom error classes
- Keep business logic server-side (no JSX in API layer)
- Write integration tests with mocked external services
- **Router logic should be application code only** - delegate to business logic services
- **Use only protected procedures** for all endpoints
- **Split procedures and routers** into separate files for better organization

### Application Logic vs Business Logic

**Application Logic** (in tRPC procedures):
- Input validation and sanitization
- Authentication and authorization checks
- Request/response formatting
- Error handling and mapping
- Delegation to business logic services
- Logging and observability

**Business Logic** (in service layer):
- Core business rules and validations
- Domain-specific operations
- Data transformations and calculations
- Business workflow orchestration
- Domain knowledge and expertise

**Example:**

```typescript
// Application Logic (tRPC Procedure)
export const createUserProcedure = protectedProcedure
  .input(createUserSchema)
  .mutation(async ({ ctx, input }) => {
    try {
      // Application logic: validation, auth, delegation
      const result = await createUser({
        userId: ctx.auth.userId,
        ...input
      });
      
      return { success: true, data: result };
    } catch (error) {
      // Application logic: error handling
      if (error instanceof CustomError) {
        throw error;
      }
      throw new CustomError("INTERNAL_ERROR", "An unexpected error occurred");
    }
  });

// Business Logic (Service Layer)
export const createUser = async (params: CreateUserParams): Promise<User> => {
  const { userId, email, name, role } = params;
  
  // Business logic: domain rules
  if (role === 'admin' && !await hasAdminPrivileges(userId)) {
    throw new CustomError("UNAUTHORIZED", "Insufficient privileges for admin role", 403);
  }
  
  // Business logic: business validations
  if (await isEmailAlreadyTaken(email)) {
    throw new CustomError("CONFLICT", "Email already registered", 409);
  }
  
  // Business logic: domain operations
  const user = await userRepo.create({
    email: email.toLowerCase(),
    name: name.trim(),
    role,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanrawlingswang/cursor-rules](https://github.com/ryanrawlingswang/cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
