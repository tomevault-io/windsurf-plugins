---
trigger: always_on
description: - **Language**: Crystal - statically typed, compiled language with Ruby-inspired syntax
---

---# CQL (Crystal Query Language) - Cursor Rules

## Language & Framework Context

### Crystal Programming Language
- **Language**: Crystal - statically typed, compiled language with Ruby-inspired syntax
- **Type System**: Leverage Crystal's compile-time type checking for type-safe database operations
- **Macros**: Use Crystal's macro system for code generation and DSL creation
- **Performance**: Crystal compiles to native code - optimize for performance at compile time
- **Memory Management**: Crystal has automatic memory management - no manual memory allocation needed

### CQL Framework Specifics
- **ORM Type**: Object-Relational Mapping library with Active Record pattern support
- **Database Support**: PostgreSQL, MySQL, SQLite through Crystal DB drivers
- **Architecture**: Supports Active Record, Repository, and Data Mapper patterns
- **Performance**: Built for high performance with compile-time optimizations

## Architecture Principles

### SOLID Principles
- **Single Responsibility**: Each model should have one reason to change
- **Open/Closed**: Extend functionality through composition and inheritance
- **Liskov Substitution**: Subclasses should be substitutable for their base classes
- **Interface Segregation**: Create focused, specific interfaces
- **Dependency Inversion**: Depend on abstractions, not concretions

### Clean Architecture
- **Domain Layer**: Core business logic in models and domain services
- **Application Layer**: Use cases and application-specific business rules
- **Infrastructure Layer**: Database, cache, external service integrations
- **Presentation Layer**: Controllers, serializers, API endpoints

## CQL-Specific Patterns

### Model Definition
```crystal
# Use clear, descriptive model names
class User
  include CQL::Model(Int64)

end
```

### Schema Organization
- **Location**: Define schemas in `src/schemas/` directory
- **Naming**: Use descriptive schema names (e.g., `AppSchema`, `BlogSchema`)
- **Structure**: Group related tables in logical schemas
- **Migrations**: Keep migrations in dedicated directories with clear naming

### Active Record Pattern
- **Inheritance**: Extend `CQL::Model(PrimaryKeyType)`
- **Validations**: Use built-in validation macros for data integrity
- **Callbacks**: Implement lifecycle callbacks for business logic
- **Relationships**: Define associations clearly with proper foreign keys

### Repository Pattern
- **Interface**: Define repository interfaces for testability
- **Implementation**: Implement concrete repositories for specific databases
- **Queries**: Encapsulate complex queries in repository methods
- **Separation**: Keep business logic separate from data access logic

## Database Best Practices

### Migrations
- **Versioning**: Use sequential numbering (001_, 002_, etc.)
- **Reversibility**: Always provide down methods for rollbacks
- **Data Safety**: Use transactions for data-changing migrations
- **Testing**: Test migrations in development environment first

### Query Optimization
- **Indexing**: Add appropriate indexes for frequently queried columns
- **N+1 Prevention**: Use eager loading for associations
- **Caching**: Implement query and fragment caching where appropriate
- **Monitoring**: Use performance monitoring tools to identify slow queries

### Schema Design
- **Normalization**: Follow database normalization principles
- **Foreign Keys**: Always define proper foreign key constraints
- **Null Handling**: Be explicit about nullable vs non-nullable fields
- **Default Values**: Set appropriate default values for columns

## Code Organization

### Directory Structure
```
src/
├── active_record/          # Active Record pattern implementation
├── cache/                  # Caching mechanisms
├── configure/              # Configuration management
├── dialects/              # Database-specific implementations
├── expression/            # Query expression building
├── performance/           # Performance monitoring
└── schemas/               # Database schema definitions

spec/
├── integration/           # Integration tests
├── patterns/             # Pattern-specific tests
├── operations/           # CRUD operation tests
└── support/              # Test support files
```

### File Naming
- **Models**: Use singular noun (e.g., `user.cr`, `post.cr`)
- **Schemas**: End with `_schema.cr` (e.g., `app_schema.cr`)
- **Migrations**: Use timestamp or sequential prefix
- **Specs**: Mirror source structure with `_spec.cr` suffix

## Performance Guidelines

### Query Performance
- **Eager Loading**: Use `includes` to prevent N+1 queries
- **Selective Loading**: Only load required columns and associations
- **Batch Operations**: Use bulk operations for multiple records
- **Connection Pooling**: Configure appropriate connection pool sizes

### Caching Strategy
- **Query Caching**: Enable per-request query caching
- **Fragment Caching**: Cache expensive query results
- **Redis Integration**: Use Redis for distributed caching
- **Cache Invalidation**: Implement proper cache invalidation strategies

### Memory Management
- **Lazy Loading**: Use lazy loading for large datasets
- **Streaming**: Process large result sets in streams
- **Object Pooling**: Reuse objects where appropriate

## Testing Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azutoolkit/cql](https://github.com/azutoolkit/cql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
