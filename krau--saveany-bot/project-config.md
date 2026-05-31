---
trigger: always_on
description: This document provides essential information for AI coding agents working on the SaveAny-Bot project.
---

# SaveAny-Bot Agent Guidelines

This document provides essential information for AI coding agents working on the SaveAny-Bot project.

## Project Overview

SaveAny-Bot is a Telegram bot written in Go that saves files/messages from Telegram and various websites to multiple storage backends (local, S3, MinIO, WebDAV, AList, Telegram). It features a plugin system for parsing web content and extensible storage backends.

**Tech Stack**: Go 1.24.2, gotd/td (Telegram MTProto), Cobra (CLI), Viper (config), GORM (ORM), SQLite, Goja (JS runtime), Playwright (browser automation)

## Build & Test Commands

### Build
```bash
# Standard build
go build -o saveany-bot .

# Run directly
go run ./cmd

# Docker build (multi-stage, Alpine-based)
docker build -t saveany-bot .
docker compose up -d
```

### Test
```bash
# Run all tests
go test ./...

# Run tests in specific package
go test ./pkg/queue
go test ./storage/telegram

# Run tests with verbose output
go test -v ./...

# Run a single test
go test -run TestQueueBasic ./pkg/queue

# Run with coverage
go test -cover ./...
```

### Lint & Format
```bash
# Format code (standard Go formatting)
go fmt ./...

# Vet code for common issues
go vet ./...

# Generate code (i18n keys)
go generate ./...
```

### Other Commands
```bash
# Update dependencies
go mod tidy

# View documentation
cd docs && hugo server -D
```

## Code Style Guidelines

### Imports
- Standard library first, then third-party, then project-internal
- Group imports with blank lines between groups
- Use explicit import aliases for clarity when needed (e.g., `storconfig`, `storenum`)

```go
import (
    "context"
    "fmt"
    
    "github.com/charmbracelet/log"
    
    "github.com/krau/SaveAny-Bot/config"
    "github.com/krau/SaveAny-Bot/pkg/enums/storage"
)
```

### Formatting
- Line length: reasonable (no hard limit, but be sensible)
- Organize code with blank lines between logical sections
- Follow standard Go conventions for braces, spacing, etc.

### Types & Interfaces
- Use clear, descriptive type names (PascalCase for exported, camelCase for unexported)
- Define interfaces where abstraction is needed (e.g., `Executable`, `StorageConfig`)
- Embed context in method signatures, not structs: `func (s *Service) Do(ctx context.Context) error`
- Prefer composition over inheritance

```go
// Interfaces define behavior
type Executable interface {
    Type() tasktype.TaskType
    Title() string
    TaskID() string
    Execute(ctx context.Context) error
}

// Structs compose behavior
type Local struct {
    config config.LocalStorageConfig
    logger *log.Logger
}
```

### Naming Conventions
- **Packages**: lowercase, single word when possible (avoid underscores)
- **Files**: lowercase with underscores for multiword (e.g., `auth_terminal.go`, `progress_reader.go`)
- **Variables**: camelCase for unexported, PascalCase for exported
- **Constants**: PascalCase for exported, camelCase for unexported (not ALL_CAPS)
- **Functions/Methods**: PascalCase for exported, camelCase for unexported
- **Test files**: `*_test.go` pattern

### Error Handling
- Always handle errors explicitly; never ignore them
- Wrap errors with context using `fmt.Errorf("context: %w", err)`
- Use `errors.Is()` and `errors.As()` for error checking
- Log errors with appropriate level (Error, Warn, Info)
- Return errors from functions rather than panicking (except for truly unrecoverable situations)

```go
// Good error handling
if err := db.Save(user).Error; err != nil {
    return fmt.Errorf("failed to save user %d: %w", user.ChatID, err)
}

// Check specific errors
if errors.Is(err, context.Canceled) {
    logger.Info("Operation was canceled")
    return nil
}
```

### Logging
- Use `github.com/charmbracelet/log` package
- Get logger from context: `log.FromContext(ctx)`
- Create prefixed loggers for components: `logger.WithPrefix("component")`
- Use appropriate levels: Debug, Info, Warn, Error
- Include context in log messages (e.g., task IDs, file names)

```go
logger := log.FromContext(ctx)
logger.Infof("Processing task: %s", task.ID)
logger.Errorf("Failed to save file %s: %v", filename, err)
```

### Concurrency
- Use channels for communication between goroutines
- Protect shared state with `sync.Mutex` or `sync.RWMutex`
- Use `sync.WaitGroup` for coordinating goroutine completion
- Always pass `context.Context` for cancellation support
- Use `context.WithCancel/WithTimeout` for managing goroutine lifetimes

```go
// Example from queue implementation
func (tq *TaskQueue[T]) Add(task *Task[T]) error {
    tq.mu.Lock()
    defer tq.mu.Unlock()
    // ... critical section
    tq.cond.Signal()
    return nil
}
```

### Comments
- Document exported types, functions, and packages with doc comments
- Start doc comments with the name being documented
- Use `//` for single-line comments
- Explain *why*, not *what* (code should be self-explanatory for "what")
- Add `[NOTE]`, `[WARN]`, `[IMPORTANT]` tags for important clarifications

```go
// GetUserByChatID retrieves a user by their Telegram chat ID.
// Returns an error if the user is not found.
func GetUserByChatID(ctx context.Context, chatID int64) (*User, error) {
```

## Architecture & Conventions

### Application Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krau/SaveAny-Bot](https://github.com/krau/SaveAny-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
