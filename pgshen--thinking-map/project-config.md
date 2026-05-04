---
trigger: always_on
description: This is a thinking map application with a Go backend (server/) and Next.js frontend (web/). The project uses modern development practices with TypeScript, shadcn/ui, Zustand, and ReactFlow for the frontend, and Go with Gin, GORM, eino and Redis for the backend.
---

# ThinkingMap Cursor Rules

## Project Overview
This is a thinking map application with a Go backend (server/) and Next.js frontend (web/). The project uses modern development practices with TypeScript, shadcn/ui, Zustand, and ReactFlow for the frontend, and Go with Gin, GORM, eino and Redis for the backend.

## Frontend Rules (web/)

### Technology Stack
- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript (strict mode)
- **UI Library**: shadcn/ui + Radix UI
- **Styling**: Tailwind CSS
- **State Management**: Zustand
- **Visualization**: ReactFlow
- **Package Manager**: pnpm
- **Icons**: Lucide React

### File Organization
- Use the established directory structure in `src/`:
  - `/app` - Next.js App Router pages
  - `/components` - Reusable UI components
  - `/features` - Business logic modules (map, panel, home, chat)
  - `/store` - Zustand state management
  - `/api` - API client and SSE handling
  - `/hooks` - Custom React hooks
  - `/types` - TypeScript type definitions
  - `/utils` - Utility functions
  - `/layouts` - Layout components

### Component Guidelines
- Use shadcn/ui components as the primary UI library
- **IMPORTANT**: Use `sonner` for toasts instead of deprecated toast components
- Follow the established component patterns in the codebase
- Use TypeScript interfaces for all props and state
- Implement proper error boundaries and loading states

### State Management
- Use Zustand for global state management
- Split stores by feature (mapStore, globalStore, etc.)
- Use TypeScript interfaces for all store types
- Implement proper selectors to avoid unnecessary re-renders
- Follow the established patterns in `/store/` directory

### API Integration
- Use the established API patterns in `/api/` directory
- Implement proper error handling and loading states
- Use SSE for real-time updates (see `use-sse-connection.ts`)
- Follow the established request/response patterns

### Styling
- Use Tailwind CSS for all styling
- Follow the established design system in `components.json`
- Use CSS variables for theming
- Implement responsive design patterns

### Code Quality
- Use ESLint and Prettier for code formatting
- Follow TypeScript strict mode guidelines
- Use proper import/export patterns
- Implement proper error handling

## Backend Rules (server/)

### Technology Stack
- **Language**: Go 1.24+
- **Framework**: Gin
- **Framework**: eino
- **ORM**: GORM with PostgreSQL
- **Cache**: Redis
- **Authentication**: JWT
- **Logging**: Zap
- **Testing**: Testify

### Project Structure
- Follow the established directory structure:
  - `/cmd` - Application entry points
  - `/internal` - Private application code
  - `/configs` - Configuration files
  - `/scripts` - Database and deployment scripts

### Code Organization
- Use proper Go package naming conventions
- Follow the established service/repository pattern
- Implement proper error handling with custom error types
- Use interfaces for dependency injection
- Follow the established middleware patterns

### Database
- Use GORM for database operations
- Follow the established model patterns
- Implement proper migrations
- Use transactions for complex operations
- Follow the established repository pattern

### API Design
- Use RESTful conventions
- Implement proper validation using the validator package
- Use DTOs for request/response handling
- Follow the established handler patterns
- Implement proper CORS configuration

### Testing
- Use the established test utilities in `/internal/service/`
- Follow the test patterns in existing test files
- Use proper test setup and teardown
- Implement integration tests for critical paths

### Configuration
- Use Viper for configuration management
- Follow the established config patterns
- Use environment-specific config files
- Implement proper validation for config values

## General Development Rules

### Git Workflow
- Use conventional commit messages
- Follow the established branching strategy
- Implement proper PR reviews
- Use meaningful commit messages

### Documentation
- Update relevant documentation when making changes
- Follow the established documentation patterns
- Use proper code comments for complex logic
- Maintain API documentation

### Error Handling
- Implement proper error handling in both frontend and backend
- Use appropriate error types and messages
- Implement proper logging
- Provide user-friendly error messages

### Performance
- Implement proper caching strategies
- Use pagination for large datasets
- Optimize database queries
- Implement proper loading states

### Security
- Implement proper authentication and authorization
- Use HTTPS in production
- Validate all user inputs
- Follow security best practices

### Testing
- Write tests for critical business logic
- Implement proper test coverage
- Use proper test data management
- Follow the established testing patterns

## Specific Patterns to Follow

### Frontend Patterns
- Use the established layout patterns (SidebarLayout, WorkspaceLayout)
- Follow the component composition patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PGshen/thinking-map](https://github.com/PGshen/thinking-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
