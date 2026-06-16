---
trigger: always_on
description: Comprehensive architectural standards and design principles following SOLID principles, Clean Architecture, and DRY practices for building maintainable, scalable software
---

# Architecture & Design Principles
# Architecture & Design Principles

<goal>
Establish comprehensive architectural standards and design principles for building maintainable, scalable, and robust software following industry best practices adapted to the project's domain-driven structure.
</goal>

## Core Architectural Principles

### SOLID Principles

<principle type="solid_srp">
**Single Responsibility Principle (SRP):**
- Each module, class, or function should have only one reason to change
- Separate business logic, data access, and presentation concerns
- Use domain-specific packages: `engine/{agent,task,tool,workflow,runtime,infra}/`
- *Implementation examples: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</principle>

<principle type="solid_ocp">
**Open/Closed Principle (OCP):**
- Open for extension, closed for modification
- Use interfaces and composition over inheritance
- Leverage factory patterns for extensible behavior
- *Factory pattern implementation: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</principle>

<principle type="solid_lsp">
**Liskov Substitution Principle (LSP):**
- Subtypes must be substitutable for their base types
- Interface implementations must honor contracts
- Ensure interface methods behave consistently
- *Interface design patterns: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</principle>

<principle type="solid_isp">
**Interface Segregation Principle (ISP):**
- Clients should not depend on interfaces they don't use
- Create small, focused interfaces
- Use interface composition for complex behavior
- *Interface composition examples: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</principle>

<principle type="solid_dip">
**Dependency Inversion Principle (DIP):**
- Depend on abstractions, not concretions
- Use dependency injection through constructors
- High-level modules should not depend on low-level modules
- *Constructor patterns: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</principle>

### DRY Principle (Don't Repeat Yourself)

<dry_strategies type="code_reuse">
**Code Reuse Strategies:**
- Extract common functionality into shared packages
- Use generic functions for similar operations
- Create utility packages for cross-cutting concerns

```go
// ✅ Good: Reusable validation utility
func ValidateRequired(value string, fieldName string) error {
    if strings.TrimSpace(value) == "" {
        return fmt.Errorf("%s is required", fieldName)
    }
    return nil
}

// Usage across multiple validators
func (v *UserValidator) ValidateName(name string) error {
    return ValidateRequired(name, "name")
}

func (v *TaskValidator) ValidateTitle(title string) error {
    return ValidateRequired(title, "title")
}
```
</dry_strategies>

<dry_strategies type="configuration_patterns">
**Configuration Patterns:**
- Centralize configuration with defaults
- Use template engine for dynamic configurations
- Avoid duplicating configuration logic
- *Configuration implementation: see [go-patterns.mdc](mdc:.cursor/rules/go-patterns.mdc)*
</dry_strategies>

### Clean Architecture

<architecture_structure type="domain_driven">
**Domain-Driven Design Structure:**
```
engine/
├── agent/     # Agent domain logic
├── task/      # Task execution domain
├── tool/      # Tool management domain
├── workflow/  # Workflow orchestration domain
├── runtime/   # Runtime execution environment
├── infra/     # Infrastructure concerns
└── core/      # Shared domain primitives
```
</architecture_structure>

<layer_separation>
**Layer Separation:**
- **Domain Layer** (`engine/core/`): Shared business entities, value objects, and cross-domain primitives
- **Application Layer** (`engine/{agent,task,tool,workflow}/`): Domain-specific business logic, use cases, and port interfaces (repositories, external services)
- **Infrastructure Layer** (`engine/infra/`): External concerns (DB, HTTP, etc.) and adapter implementations
- **Runtime Layer** (`engine/runtime/`): Execution environment and system orchestration

**Interface Ownership Clarification:**
- **Port Interfaces** (e.g., Repository, ExternalService): Defined in Application Layer packages where they're used
- **Domain Entities**: Defined in Domain Layer (`engine/core/`) for cross-domain sharing
- **Adapter Implementations**: Defined in Infrastructure Layer, implementing Application Layer interfaces
</layer_separation>

<dependency_flow>
```go
// ✅ Good: Dependencies flow inward
package task

import (
    "context"
    "github.com/project/engine/core" // Domain entities
)

type Service struct {
    repo Repository // Interface defined in domain
}

type Repository interface { // Domain-defined interface
    Save(ctx context.Context, task *core.Task) error
    Find(ctx context.Context, id core.ID) (*core.Task, error)
}

// Implementation in infrastructure layer
package infra

import (
    "github.com/project/engine/task" // Application layer
)

type PostgreSQLTaskRepository struct {
    db *sql.DB
}

func (r *PostgreSQLTaskRepository) Save(ctx context.Context, task *core.Task) error {
    // Implementation details
}
```
</dependency_flow>

### Clean Code Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
