---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
```bash
npm run dev                # Start development server
npm run dev:setup         # Run development setup (installs deps, generates Prisma, seeds DB)
./test-website.sh         # Full stack website testing script
```

### Type Checking & Linting
```bash
npm run type-check        # Quick TypeScript check
npm run type-check:full   # Full type check (use before commits)
npm run type-check:watch  # Continuous type checking
npm run lint              # Run ESLint
npm run lint:fix          # Auto-fix ESLint issues
npm run precommit         # Run both type-check:full and lint (use before commits)
```

### Building & Production
```bash
npm run build             # Build for production
npm run build:vercel      # Build with increased memory for Vercel
npm run start             # Start production server
npm run production:ready  # Full production validation
```

### Database
```bash
npm run db:generate       # Generate Prisma client
npm run db:push          # Push schema changes
npm run db:migrate       # Run migrations
npm run db:migrate:deploy # Deploy database migrations
npm run db:studio        # Open Prisma Studio GUI
npm run db:seed          # Seed database
npm run db:reset         # Reset database (CAUTION: destroys data!)
```

### Testing
```bash
npm run test             # Run Jest tests
npm run test:coverage    # Test coverage report
npm run test:watch       # Watch mode
npm run test:e2e         # Playwright E2E tests
npm run test:e2e:ui      # Playwright UI mode
npm run test:unit        # Unit tests only
npm run test:integration # Integration tests only
npm run test:components  # Component tests only
npm run test:performance # Performance tests only
npm run test:multi-tenant # Multi-tenant tests only
npm run test:all         # All test suites
npm run test:auth-flow   # Test authentication flow manually
npm run test:full-stack  # Full stack testing

# Run a single test file
npm test -- path/to/test.spec.ts

# Test shell scripts
./test-website.sh        # Full stack website testing
```

### Performance & Security
```bash
npm run performance:analyze  # Bundle analysis
npm run security:check      # Security configuration check
npm run security:audit      # Full security audit
npm run audit:security      # NPM security audit
npm run audit:fix           # Auto-fix security vulnerabilities
```

### Development Utilities
```bash
npm run verify           # Full verification: generate, type-check, lint, build
npm run verify:quick     # Quick verification: generate and lint only
npm run clean            # Clean Next.js build cache
npm run clean:all        # Full clean including node_modules
npm run generate-secrets # Generate secure secrets
npm run debug:login      # Debug login issues
npm run setup            # Environment setup
```

### Documentation
```bash
npm run docs:generate    # Generate API documentation
npm run docs:serve      # Serve docs on port 8080
```

## Architecture Overview

### Project Structure
- **Framework**: Next.js 15 App Router with React 18
- **Language**: TypeScript (strict mode currently disabled)
- **Database**: PostgreSQL via Prisma ORM (Supabase hosted)
- **Authentication**: NextAuth.js v4 with Google OAuth and credentials providers (fully consolidated)
- **Styling**: Tailwind CSS with custom enterprise design system
- **State Management**: React Context API and Zustand
- **Caching**: Redis with multi-layer caching strategy
- **AI Integration**: OpenAI GPT-4 and Anthropic Claude

### Key Directories
- `/src/app/` - Next.js App Router pages and API routes
  - `/api/` - RESTful API endpoints with standardized middleware
  - Page routes organized by feature (dashboard, compliance, etc.)
- `/src/components/` - React components organized by domain
  - `/ui/` - Reusable UI primitives
  - Feature-specific components (risks, compliance, ai, etc.)
- `/src/lib/` - Core utilities and configurations
  - `/api/` - API standardization system (MUST use for new endpoints)
  - `/security/` - Security middleware
  - `/performance/` - Optimization utilities
  - `/auth/` - Authentication utilities and middleware
  - `/billing/` - Billing and subscription management
- `/src/context/` - Global state management providers
- `/src/services/` - Business logic service layer
  - `/ai/` - AI service integrations
- `/src/hooks/` - Custom React hooks
- `/src/types/` - TypeScript type definitions
- `/scripts/` - Development and deployment scripts
- `/src/scripts/` - Test and utility scripts

### Critical Development Guidelines

1. **API Development**: Always use `withApiMiddleware()` wrapper for new API endpoints
   ```typescript
   // Current pattern (most endpoints use this):
   export const POST = withApiMiddleware(async (req) => {
     const user = (req as any).user;
     // Your handler code
     return NextResponse.json({ data });
   });
   
   // Recommended pattern (newer, supports validation & rate limiting):
   export const POST = withApiMiddleware({
     requireAuth: true,
     bodySchema: MyBodySchema,
     rateLimiters: ['standard'] // or 'auth', 'fileUpload', 'expensive', 'bulk', 'report'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jungkj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
