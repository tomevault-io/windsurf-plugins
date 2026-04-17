---
trigger: always_on
description: You are an AI assistant working in the LeadSend monorepo. Follow these rules strictly to maintain code quality and consistency.
---

# LeadSend Code Standards & Cursor Rules

You are an AI assistant working in the LeadSend monorepo. Follow these rules strictly to maintain code quality and consistency.

## Project Context

This is an Nx monorepo with multiple NestJS backend services, React frontend applications, and shared libraries. The main applications are:

- `apps/api/` - Main API service
- `apps/scheduler-service/` - Scheduling service
- `apps/worker-service/` - Background worker service
- `apps/web/` - React frontend application
- `library/` - Shared libraries (storage, queue-v2, workflow-engine, frontend-shared)

## General Rules

### Code Style

- **Always use TypeScript** - No JavaScript files
- Follow existing ESLint and Prettier configurations
- Use meaningful, descriptive variable and function names
- Keep functions small and focused on a single task
- Write clean, well-documented code with JSDoc for public APIs

### File Organization

- Use kebab-case for file names (e.g., `workflow-generation.service.ts`)
- Group related files in directories by feature
- Import structure: external imports first, then internal imports
- Always check existing patterns before creating new files

## Backend Standards (NestJS)

### Architecture Patterns

- Follow NestJS module structure strictly
- Use dependency injection for all services
- Organize code by feature modules, not by file type
- Follow RESTful API design principles

### Module Structure

Every feature module should have:

```typescript
@Module({
  imports: [ExternalModule], // External dependencies first
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService], // Only if needed by other modules
})
export class FeatureModule {}
```

### Controller Patterns

```typescript
@ApiTags('Features') // Always add Swagger tags
@Controller('features')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new feature' })
  @ApiResponse({ status: 201, description: 'Feature created.' })
  async create(@Body() createFeatureDto: CreateFeatureDto) {
    return this.featureService.create(createFeatureDto);
  }
}
```

### Service Patterns

```typescript
@Injectable()
export class FeatureService {
  constructor(private readonly repository: Repository) {}

  async create(data: CreateFeatureDto) {
    // Business logic goes here
    return this.repository.create(data);
  }
}
```

### DTO Validation

Always create DTOs with proper validation:

```typescript
export class CreateFeatureDto {
  @ApiProperty({ description: 'Feature name', example: 'New Feature' })
  @IsNotEmpty()
  @IsString()
  name: string;

  @ApiProperty({ description: 'Optional description', required: false })
  @IsOptional()
  @IsString()
  description?: string;
}
```

## Shared Libraries Usage

### Storage Library (`@leadsend/storage`)

- Use for all database entities and repositories
- Import entities from `@leadsend/storage`
- Never create database entities outside this library

### Queue Library (`@leadsend/queue-v2`)

- Use for all message queuing operations
- Follow existing queue patterns and interfaces
- Use WorkflowManager for workflow-related queuing

### Workflow Engine (`@leadsend/workflow-engine`)

- Use for workflow processing logic
- Import workflow interfaces and services from this library

## Database Standards

### Entity Definitions

- Define all entities in `@leadsend/storage`
- Use snake_case for table and column names
- Tables should be plural (e.g., `workflow_executions`)
- Columns should be descriptive (e.g., `created_at`, `workflow_id`)

### Repository Usage

- Use TypeORM repositories through the storage library
- Handle all database operations in services, never in controllers
- Use migrations for schema changes

## API Design

### RESTful Endpoints

- Use resource-based URLs (nouns, not verbs)
- Follow HTTP method conventions:
  - GET: Retrieve resources
  - POST: Create resources
  - PUT: Update entire resources
  - PATCH: Partial updates
  - DELETE: Remove resources

### Swagger Documentation

- Add `@ApiTags()` to all controllers
- Use `@ApiOperation()` for endpoint descriptions
- Add `@ApiResponse()` for different response scenarios
- Document all DTOs with `@ApiProperty()`

## Error Handling

### Exception Handling

- Use NestJS built-in HTTP exceptions
- Log errors with appropriate context
- Never expose sensitive information in error responses
- Use proper HTTP status codes

### Validation

- Always validate input with DTOs and class-validator
- Use appropriate validation decorators (@IsNotEmpty, @IsString, etc.)
- Provide meaningful error messages

## Security Standards

### Authentication & Authorization

- Use existing auth guards from shared libraries
- Apply `@AllowUnauthorizedRequest()` only for public endpoints
- Never bypass authentication without good reason
- Validate all user inputs

### Data Protection

- Use parameterized queries (TypeORM handles this)
- Never log sensitive information
- Validate file uploads and user inputs
- Set proper CORS settings

## Frontend Standards (React)

### Component Architecture

- Use function components with hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parthvsquare) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
