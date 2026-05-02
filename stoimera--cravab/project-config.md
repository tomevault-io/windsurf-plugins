---
trigger: always_on
description: Multi-tenant AI-powered business management PWA with Next.js 15, Supabase, Vapi AI integration, and comprehensive caching architecture.
---

# Cravab - Comprehensive Cursor IDE Rules

## Project Context
Multi-tenant AI-powered business management PWA with Next.js 15, Supabase, Vapi AI integration, and comprehensive caching architecture.

## Architecture & Patterns

### Multi-tenant Architecture
- All data operations must include `tenant_id` isolation
- Use `user.user_metadata?.tenant_id || user.id` for tenant identification
- Implement proper RLS (Row Level Security) policies for data isolation
- Never expose cross-tenant data
- Never use hardcoded fallbacks that could cause tenant confusion
- Always validate tenant context before any operation
- Ensure complete tenant data isolation at all levels
- Use tenant-specific configuration for all settings
- Implement proper tenant validation and error handling

### PWA-First Design
- Implement offline-first patterns with proper cache invalidation
- Use unified `CacheInvalidationService` for all cache operations
- Implement proper service worker caching
- Use PWA storage with lazy initialization to prevent SSR issues

### AI Integration
- Follow Vapi webhook integration patterns
- Use proper webhook verification for external integrations
- Implement proper error handling for AI service failures
- Follow the established system prompt patterns
- Use `WebhookMonitor` for AI call performance tracking
- Implement proper function call parameter mapping with `mapVapiParameters()`
- Use proper status mapping with `mapVapiStatusToDbStatus()`
- Follow the comprehensive Vapi system prompt patterns
- Implement proper service area validation before booking
- Use proper client lookup before creating new clients
- Implement proper business hours validation
- Use proper date handling with tenant-specific timezone considerations
- Use `getTenantTimezone()` to get tenant's configured timezone
- Require tenant timezone to be configured - no fallback timezone
- Use `parseRelativeDate()` for handling relative dates in tenant timezone
- Use `handleTenantTimezoneDateTime()` for datetime parsing
- Follow the mandatory function call order: getCurrentDate → checkServiceArea → findServiceForClient → bookAppointment

## Code Standards

### TypeScript & Validation
- Use strict TypeScript with proper interfaces from `database-comprehensive.ts`
- Validate all API inputs with Zod schemas from `schemas.ts`
- Never use `any` types - provide specific interfaces
- Use proper error handling with `StandardError` types
- Follow the comprehensive schema patterns

### API Routes (Next.js 15 App Router)
```typescript
export async function GET(request: NextRequest) {
  try {
    const cookieStore = await cookies()
    const supabase = createClient(cookieStore)
    const { data: { user } } = await supabase.auth.getUser()
    
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }
    
    const tenantId = user.user_metadata?.tenant_id || user.id
    // ... implementation
  } catch (error) {
    console.error('Error:', error)
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

### React Components
- Use Radix UI components with proper accessibility
- Implement proper loading states and error boundaries
- Follow the animation system patterns with Framer Motion
- Use proper form validation with React Hook Form + Zod
- Implement proper PWA patterns for offline functionality

### Database Operations
- Use `DatabaseService` class for all database operations
- Implement proper RLS (Row Level Security) policies
- Use proper error handling and transaction management
- Follow the comprehensive schema patterns

### Caching Strategy
- Use unified `CacheInvalidationService` for all cache operations
- Implement proper TTL values: 30s (real-time), 5min (frequent), 15min (stable), 1h (static)
- Use React Query for server state management
- Implement proper offline cache with PWA storage

## Visual Design Principles

### Design Standards
- **NO EMOJIS** - Never use emojis in code, UI, or comments
- **Mobile-first design** - All layouts must be optimized for mobile devices first
- **Light theme only** - White background, black text, blue accents
- **Clean, polished, minimalistic, professional appearance** - Business-focused UI

### Performance Optimization
- Use proper React optimization patterns (memo, useMemo, useCallback)
- Implement proper code splitting and lazy loading
- Use proper image optimization with Next.js Image
- Follow the performance monitoring patterns
- Use skeleton loaders for data fetching states
- Use `DatabaseOptimizer` for query optimization and caching
- Implement proper query metrics tracking
- Use `WebhookMonitor` for performance monitoring
- Set performance thresholds: maxResponseTime: 5000ms, maxMemoryUsage: 100MB, maxErrorRate: 10%
- Implement proper cache hit rate monitoring (target: >50%)
- Use `CacheManager` for centralized cache management

## Security & Authentication

### Security Standards
- Encrypt sensitive data (API keys) with proper encryption using `EncryptionService`
- Implement proper input validation and sanitization using `InputValidator`
- Use proper authentication and authorization patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stoimera/Cravab](https://github.com/stoimera/Cravab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
