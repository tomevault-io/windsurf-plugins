---
trigger: always_on
description: Usernaut is a Kubernetes operator built using Go 1.24.2 and controller-runtime. It manages user and team synchronization across multiple backends (LDAP, Fivetran, Red Hat Rover) with caching capabilities.
---

# GitHub Copilot Instructions for Usernaut

## Project Overview

Usernaut is a Kubernetes operator built using Go 1.24.2 and controller-runtime. It manages user and team synchronization across multiple backends (LDAP, Fivetran, Red Hat Rover) with caching capabilities.

## Go Development Best Practices

### Code Style and Formatting

- **Always run `gofmt`** on all Go files before committing
- Use `goimports` to automatically manage imports
- Follow the existing code structure and naming conventions
- Use camelCase for variable and function names, PascalCase for exported types
- Keep line length under reasonable limits (current project uses `lll` linter)

### Error Handling

- Always handle errors explicitly - never ignore them
- Use descriptive error messages that include context
- Wrap errors with additional context using `fmt.Errorf` or error wrapping
- Return errors as the last return value in functions
- Use early returns to reduce nesting

```go
// Good
func processUser(userID string) (*User, error) {
    if userID == "" {
        return nil, errors.New("userID cannot be empty")
    }

    user, err := fetchUser(userID)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch user %s: %w", userID, err)
    }

    return user, nil
}
```

### Struct and Interface Design

- Use composition over inheritance
- Keep interfaces small and focused (interface segregation principle)
- Define interfaces where they are used, not where they are implemented
- Use struct embedding for extending functionality
- Add JSON/YAML tags for serialization when needed

```go
// Good interface design
type UserFetcher interface {
    FetchUser(ctx context.Context, id string) (*User, error)
}

type TeamManager interface {
    CreateTeam(ctx context.Context, team *Team) error
    DeleteTeam(ctx context.Context, teamID string) error
}
```

### Context Usage

- Always pass `context.Context` as the first parameter to functions that may block
- Use `context.Background()` for main functions and tests
- Use `context.WithTimeout()` or `context.WithCancel()` for operations with timeouts
- Never store context in structs

### Testing

- Write unit tests for all public functions
- Use table-driven tests for multiple test cases
- Mock external dependencies using interfaces
- Use meaningful test names that describe the scenario
- Follow the AAA pattern: Arrange, Act, Assert

```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name    string
        input   *User
        want    *User
        wantErr bool
    }{
        {
            name:  "valid user creation",
            input: &User{Email: "test@example.com"},
            want:  &User{ID: "123", Email: "test@example.com"},
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

### Logging

- Use structured logging with logrus (as used in the project)
- Include relevant context fields in log entries
- Use appropriate log levels (Debug, Info, Warn, Error)
- Log errors with sufficient context but avoid logging the same error multiple times

```go
log := logger.Logger(ctx).WithFields(logrus.Fields{
    "userID": userID,
    "component": "user-service",
})
log.Info("processing user")
```

### Package Organization

- Keep packages focused and cohesive
- Use internal packages for implementation details
- Group related functionality together
- Avoid circular dependencies
- Use clear, descriptive package names

### Performance Considerations

- Use context for cancellation and timeouts
- Implement proper caching strategies (as done with Redis in this project)
- Avoid unnecessary allocations in hot paths
- Use buffered channels appropriately
- Consider using sync.Pool for frequently allocated objects

### Kubernetes Operator Specific Guidelines

- Follow controller-runtime patterns for reconciliation
- Use proper status updates with conditions
- Implement proper finalizers for cleanup
- Handle resource creation idempotently
- Use client-go best practices for API interactions

```go
// Good reconciler pattern
func (r *GroupReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := r.Log.WithValues("group", req.NamespacedName)

    var group usernautdevv1alpha1.Group
    if err := r.Get(ctx, req.NamespacedName, &group); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // Reconciliation logic here

    return ctrl.Result{}, nil
}
```

### Security Best Practices

- Never log sensitive information (passwords, API keys, tokens)
- Validate all inputs
- Use proper authentication and authorization
- Handle secrets securely
- Implement proper RBAC for Kubernetes resources

### Code Quality Tools

This project uses golangci-lint with the following enabled linters:

- `dupl` - Check for code duplication
- `errcheck` - Check for unchecked errors
- `copyloopvar` - Check for loop variable copying issues
- `ginkgolinter` - Ginkgo test framework linting
- `goconst` - Check for repeated strings that could be constants
- `gocyclo` - Check cyclomatic complexity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-data-and-ai/usernaut](https://github.com/redhat-data-and-ai/usernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
