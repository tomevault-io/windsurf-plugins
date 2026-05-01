---
trigger: always_on
description: This document provides comprehensive guidance for Go development in Crescendo Lab, covering architecture patterns, coding style, and best practices.
---

# Go Development Guide in CL

This document provides comprehensive guidance for Go development in Crescendo Lab, covering architecture patterns, coding style, and best practices.

## Table of Contents
- [Architecture Patterns](mdc:#architecture-patterns)
- [Package Structure](mdc:#package-structure)
- [Code Organization](mdc:#code-organization)
- [Naming Conventions](mdc:#naming-conventions)
- [Types and Structs](mdc:#types-and-structs)
- [Functions and Methods](mdc:#functions-and-methods)
- [Error Handling](mdc:#error-handling)
- [Concurrency](mdc:#concurrency)
- [Testing](mdc:#testing)
- [Performance Considerations](mdc:#performance-considerations)
- [Logging and Observability](mdc:#logging-and-observability)
- [Documentation](mdc:#documentation)
- [Project Structure](mdc:project-structure)

## Architecture Patterns

### Clean Architecture

Go project in CL follows a clean architecture pattern with distinct layers:

1. **Domain Layer** (`internal/domain/`)
   - Contains business entities and logic
   - Independent of external frameworks and databases
   - Defines interfaces that are implemented by outer layers

2. **Application Layer** (`internal/app/`)
   - Contains application services that orchestrate domain entities
   - Implements use cases of the system
   - Depends on domain layer, but not on external frameworks

3. **Adapter Layer** (`internal/adapter/`)
   - Implements interfaces defined in the domain layer
   - Connects the application to external systems (databases, message brokers, etc.)
   - Contains concrete implementations of repositories and services

4. **Router Layer** (`internal/router/`)
   - Entry points to the application (HTTP handlers, CLI commands)
   - Depends on application services
   - Translates external requests to internal application calls

### Common Patterns

#### Interface Design
- Define interfaces in the layer that uses them, not in the layer that implements them
- Keep interfaces small and focused on a single responsibility
- Use dependency injection to provide implementations

```go
// In domain layer
type MessageRepository interface {
    WriteMessageBatch(context.Context, []message.Message) common.Error
    GetMessage(ctx context.Context, channelType message.ChannelType, externalMessageID string) (*message.Message, common.Error)
}

// In adapter layer
type messageRepository struct {
    // implementation details
}
```

#### Service Pattern
- Services should be stateless when possible
- Use parameter structs for service constructors with many dependencies
- Services should depend on interfaces, not concrete implementations

```go
type GatewayServiceParam struct {
    MessageRepository   MessageRepository
    ReportRepository    ReportRepository
    // other dependencies
}

func NewGatewayService(ctx context.Context, param GatewayServiceParam) *GatewayService {
    return &GatewayService{
        messageRepository: param.MessageRepository,
        // initialize other dependencies
    }
}
```

#### Repository Pattern
- Repositories abstract data access logic
- They should return domain entities, not data transfer objects
- Use custom error types for domain-specific errors

#### Factory Pattern
- Use factories to create complex domain entities or services
- Hide implementation details behind factory interfaces

```go
type ChannelFactory interface {
    Channel(context.Context, message.ChannelType) (message.Channel, common.Error)
}
```

#### Event-Driven Architecture
- Use event brokers for asynchronous communication between services
- Define clear event interfaces and payloads
- Use event-driven patterns for scalability and loose coupling

## Package Structure
- Package names should be short, concise, and lowercase (e.g., `message`, `gateway`, `common`)
- One package per directory
- Package name should match the directory name
- Use `internal` directory for code that should not be imported by other projects

## Code Organization

### Package Design
- Keep packages focused on a single responsibility
- Avoid circular dependencies between packages
- Organize code by domain concept, not by technical function

### File Organization
- Keep files to a reasonable size (under 500 lines if possible)
- Group related functionality in the same file
- Place interfaces in the same package as the code that uses them
- Use separate files for tests

### Imports
- Group imports into standard library, external packages, and internal packages
- Within each group, imports should be alphabetically sorted
- Use blank lines to separate import groups

```go
import (
    "context"
    "encoding/json"
    "time"

    "github.com/rs/zerolog"
    "github.com/alecthomas/kingpin/v2"

    "github.com/chatbotgang/medley/internal/domain/message"
)
```

## Naming Conventions
- Use `CamelCase` for exported names (public)
- Use `mixedCase` for non-exported names (private)
- Use acronyms consistently (e.g., `ID`, `URL`, `HTTP`)
- Constants should use `PascalCase` with descriptive prefixes
- Interface names should not end with `-er` (e.g., `MessageRepository` not `MessageRepositorier`)

## Types and Structs
- Define types at the top of the file, followed by constants, then variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chatbotgang/ai-coding-workshop-250712](https://github.com/chatbotgang/ai-coding-workshop-250712) — distributed by [TomeVault](https://tomevault.io/claim/chatbotgang).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
