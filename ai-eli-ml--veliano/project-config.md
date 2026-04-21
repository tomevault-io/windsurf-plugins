---
trigger: always_on
description: Development standards and practices
---


 # Development Standards and Best Practices

This document outlines our development standards and best practices to maintain code quality and consistency.

## Next.js 15 Specific Standards

- Use the App Router for all new route development
- Implement Server Components by default, only use Client Components when necessary
- Use Server Actions for form submissions and data mutations
- Follow proper async/await patterns for Web APIs (cookies, headers)
- Use middleware for authentication and protection patterns

## TypeScript Standards

- Use TypeScript for all new code
- Define explicit types for function parameters and return values
- Use interfaces for complex objects and type aliases for simple types
- Avoid `any` type wherever possible
- Use proper type imports and exports

## Authentication Testing Standards

- Use mock Supabase client for authentication tests
- Implement proper test doubles for Supabase responses
- Test all authentication flows:
  - Sign up
  - Sign in
  - Password reset
  - Session management
  - Error cases
- Use proper TypeScript types for test data
- Follow AAA pattern (Arrange, Act, Assert)
- Implement proper cleanup in test teardown
- Use environment variables for test configuration

## Supabase Integration

- Use the repository pattern for database access
- Follow Row Level Security (RLS) policies for data protection
- Create proper TypeScript types for database schema
- Use Server Components for database queries
- Implement proper error handling for database operations

## Code Organization

- Use functional components instead of class components
- Follow the single responsibility principle
- Keep components small and focused
- Use custom hooks for shared logic
- Separate business logic from UI components

## Performance Optimization

- Implement proper loading states for async operations
- Use image optimization for all images
- Implement code splitting with dynamic imports
- Minimize client-side JavaScript
- Use proper caching strategies for static content

## Deployment

- Use Vercel for deployment
- Set up proper environment variables
- Implement pre-deployment checks
- Follow the deployment checklist in `.cursor/DEPLOY.md`
- Monitor performance after deployment

# Development Standards

## Current Focus Areas

### 1. Performance Monitoring
- Implement comprehensive monitoring for all critical paths
- Track Core Web Vitals (LCP, FID, CLS)
- Monitor API and database performance
- Ensure proper error tracking and reporting
- Maintain type safety in monitoring implementations

### 2. Type Safety
- Zero tolerance for `any` types
- Explicit type definitions for all functions
- Proper error type handling
- Type-safe API responses
- Validated form inputs with type checking

### 3. Error Handling
- Implement error boundaries at appropriate levels
- Track errors with proper context
- Handle both client and server errors gracefully
- Provide user-friendly error messages
- Log errors with appropriate severity levels

## Code Quality Requirements

### TypeScript Standards
- Strict mode enabled
- No implicit any
- No unsafe type assertions
- Proper generic type usage
- Interface-first approach
- Proper error type definitions

### Component Standards
- Functional components only
- Props interface definitions
- Error boundary implementation
- Loading state handling
- Proper type checking

### Testing Requirements
- Unit tests for utilities
- Integration tests for flows
- Error case coverage
- Performance monitoring tests
- Type checking in tests

## Performance Standards

### Monitoring Requirements
- Core Web Vitals tracking
- API performance monitoring
- Database query monitoring
- Error tracking and reporting
- Performance regression testing

### Optimization Requirements
- Code splitting
- Image optimization
- Font optimization
- API response caching
- Database query optimization

## Documentation Requirements

### Code Documentation
- Interface documentation
- Function documentation
- Error handling documentation
- Performance considerations
- Type definitions

### API Documentation
- OpenAPI/Swagger specs
- Error responses
- Type definitions
- Performance characteristics
- Security considerations

## Current Phase Standards (Phase 2 → 3 Transition)

### Code Quality Requirements

1. TypeScript
   - Strict mode enabled
   - No `any` types in core functionality
   - Proper type exports for all components
   - Type-safe database operations
   - Comprehensive interface definitions

2. Component Structure
   - Server Components by default
   - Client components only when necessary
   - Proper error boundaries
   - Loading state handling
   - Optimistic updates where appropriate

3. Testing Requirements
   - Unit tests for all components
   - Integration tests for user flows
   - RLS policy tests
   - Error handling tests
   - Performance tests

### Database Standards

1. Supabase Integration
   - Type-safe queries using generated types
   - RLS policies for all tables
   - Proper error handling
   - Optimistic updates with fallbacks
   - Transaction support where needed

2. Schema Management
   - Migrations for all changes
   - Proper indexing
   - Foreign key constraints
   - Cascade behaviors defined
   - Backup strategies

### Error Handling

1. Client-Side

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ai-Eli-ML) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
