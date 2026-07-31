---
trigger: always_on
description: - **MANDATORY**: Organize code by business domains, not technical layers
---

---
applyTo: '**'
---

# Anubis Project AI Development Rules

## Core Architecture Principles

### Domain-Driven Design (DDD)

- **MANDATORY**: Organize code by business domains, not technical layers
- **Structure**: `/src/task-workflow/domains/{domain-name}/` for each business domain
- **Separation**: Keep core business services separate from MCP interface tools
- **Example**: Core services in `core-workflow/` exported to MCP tools in `workflow-rules/`

### Module Organization

- **NestJS Modules**: One module per domain with clear imports/exports
- **Dependency Injection**: Use NestJS DI container, avoid manual instantiation
- **Module Exports**: Export only what other modules need to consume
- **Clean Imports**: Import PrismaModule in domains that need database access

### Service Layer Architecture

- **Service Responsibility**: One service per business capability
- **Naming Convention**: `{Domain}OperationsService` for core business logic
- **MCP Separation**: Keep MCP tools thin, delegate to operation services
- **Service Dependencies**: Inject dependencies via constructor, use readonly

## TypeScript & Code Quality

### Type Safety Rules

- **Strict Mode**: Always use TypeScript strict mode settings
- **Explicit Types**: Prefer explicit return types for public methods
- **No Any**: Avoid `any` type unless absolutely necessary (ESLint rule enabled)
- **Null Safety**: Use strictNullChecks, handle null/undefined explicitly
- **Interface First**: Define interfaces before implementation

### Code Style & Formatting

- **Prettier**: Use configured Prettier for consistent formatting
- **ESLint Rules**: Follow configured TypeScript ESLint rules
- **Unused Variables**: Prefix unused parameters with `_` (ESLint rule)
- **Import Organization**: Group imports: Node modules → NestJS → Internal → Relative

### Error Handling

- **Global Exception Filter**: Use established GlobalExceptionFilter pattern
- **Prisma Errors**: Use prisma-error.handler for database error translation
- **Logging**: Use NestJS Logger with structured logging format
- **MCP Debugging**: Include context and timestamp in MCP error logs

## Database & Prisma Best Practices

### Schema Organization

- **Multi-file Schema**: Separate models by concern in .prisma files
- **Naming**: Use descriptive file names (task-models.prisma, rule-models.prisma)
- **Enums**: Keep all enums in workflow-enums.prisma
- **Generated Output**: Maintain custom output path in generated/prisma

### Prisma Service Patterns

- **PrismaService**: Extend PrismaClient in dedicated service
- **Transaction Scope**: Use Prisma transactions for multi-operation consistency
- **Type Generation**: Run `prisma generate` after schema changes
- **Migration**: Use `prisma migrate` for schema evolution

### Database Operations

- **Repository Pattern**: Avoid raw Prisma calls in controllers/tools
- **Query Optimization**: Use Prisma's include/select for efficient queries
- **Error Handling**: Wrap Prisma operations with error boundary
- **Connection Management**: Let Prisma handle connection pooling

## MCP Integration Patterns

### Tool vs Service Separation

- **Core Services**: Business logic in operation services (no MCP dependencies)
- **MCP Tools**: Thin layer that calls operation services
- **Data Flow**: MCP Tool → Operation Service → Prisma Service
- **Error Boundary**: Handle MCP-specific errors at tool level

### MCP Configuration

- **Transport Types**: Support STDIO, SSE, and STREAMABLE_HTTP transports
- **Environment Config**: Use environment variables for MCP configuration
- **Tool Registration**: Register tools through @rekog/mcp-nest decorators
- **Validation**: Use Zod schemas for MCP tool parameter validation

### Performance & Caching

- **MCP Cache**: Use mcp-cache.service for expensive operations
- **Performance Monitor**: Apply @Performance decorator to critical methods
- **Memory Management**: Clean up resources in MCP tool handlers
- **Request Scoping**: Keep MCP tools stateless when possible

## File Organization & Naming

### Directory Structure

```
src/
├── task-workflow/
│   ├── domains/
│   │   ├── {domain}/
│   │   │   ├── {domain}.module.ts
│   │   │   ├── {operation}.service.ts
│   │   │   └── schemas/
│   ├── types/
│   ├── utils/
│   └── config/
```

### Naming Conventions

- **Files**: kebab-case for files (`task-operations.service.ts`)
- **Classes**: PascalCase with descriptive suffixes (`TaskOperationsService`)
- **Methods**: camelCase with verb prefixes (`createTask`, `findTaskById`)
- **Constants**: SCREAMING_SNAKE_CASE for module-level constants
- **Interfaces**: PascalCase with `I` prefix for complex interfaces

## Testing Strategy

### Unit Testing

- **Jest Configuration**: Use configured Jest with ts-jest transformer
- **Test Location**: Tests alongside source files with `.spec.ts` suffix
- **Mocking**: Mock Prisma operations using jest-mock-extended
- **Coverage**: Aim for high coverage on business logic services

### Integration Testing

- **Database Testing**: Use test database for Prisma integration tests
- **MCP Testing**: Test MCP tools end-to-end with mock transports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hive-Academy/Anubis-MCP](https://github.com/Hive-Academy/Anubis-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
