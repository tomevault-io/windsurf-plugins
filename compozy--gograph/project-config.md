---
trigger: always_on
description: Core Go patterns and conventions required for Compozy development
---

# Core Go Patterns & Conventions

## Concurrency Patterns

<pattern type="thread_safe_structures">
```go
// Thread-safe structs with embedded mutex
type Status struct {
    Name   string
    mu     sync.RWMutex // Protects all fields
}
```
</pattern>

<pattern type="concurrent_operations">
```go
// Concurrent operations with errgroup
g, ctx := errgroup.WithContext(ctx)
for _, item := range items {
    item := item // capture loop variable
    g.Go(func() error { return process(ctx, item) })
}
return g.Wait()
```
</pattern>

## Factory Pattern (OCP Implementation)

<requirement type="mandatory">
**MANDATORY for service creation:**
</requirement>

<pattern type="factory_implementation">
```go
// ✅ Good: Extensible through interfaces
type Storage interface {
    Save(ctx context.Context, data []byte) error
}

type StorageFactory struct{}
func (f *StorageFactory) CreateStorage(storageType string) (Storage, error) {
    switch storageType {
    case "redis": return NewRedisStorage(), nil
    case "memory": return NewMemoryStorage(), nil
    default: return nil, fmt.Errorf("unsupported storage type: %s", storageType)
    }
}

// Usage in constructors
func NewStorage(config *StorageConfig) (Storage, error) {
    factory := &StorageFactory{}
    return factory.CreateStorage(config.Type)
}
```
</pattern>

## Configuration with Defaults

<requirement type="always">
**Always provide defaults:**
</requirement>

<pattern type="configuration_defaults">
```go
func NewService(config *Config) *Service {
    if config == nil {
        config = DefaultConfig() // Always provide defaults
    }
    return &Service{config: config}
}
```
</pattern>

<pattern type="configuration_implementation">
```go
// ✅ Good: Centralized configuration with defaults
type ServiceConfig struct {
    Port        int           `yaml:"port"`
    Timeout     time.Duration `yaml:"timeout"`
    MaxRetries  int           `yaml:"max_retries"`
}

func DefaultServiceConfig() *ServiceConfig {
    return &ServiceConfig{
        Port:       8080,
        Timeout:    30 * time.Second,
        MaxRetries: 3,
    }
}

func NewServiceFromConfig(cfg *ServiceConfig) *Service {
    if cfg == nil {
        cfg = DefaultServiceConfig()
    }
    return &Service{config: cfg}
}
```
</pattern>

## Graceful Shutdown

<requirement type="long_running_services">
**REQUIRED for long-running services:**
</requirement>

<pattern type="graceful_shutdown">
```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
select {
case <-ctx.Done():
    return shutdown(ctx)
case <-quit:
    return shutdown(ctx)
}
```
</pattern>

## Middleware Pattern

<context type="http_handlers">
**For HTTP handlers:**
</context>

<pattern type="middleware_implementation">
```go
func authMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        if !isValidToken(c.GetHeader("Authorization")) {
            c.JSON(401, gin.H{"error": "unauthorized"})
            c.Abort()
            return
        }
        c.Next()
    }
}
```
</pattern>

## Resource Management

<context type="connection_handling">
**Connection limits and cleanup:**
</context>

<pattern type="resource_management">
```go
// Connection limits
if len(m.clients) >= m.config.MaxConnections {
    return fmt.Errorf("max connections reached")
}

// Cleanup with defer
defer func() {
    m.cancel()
    m.wg.Wait()
    if closeErr := m.conn.Close(); closeErr != nil {
        log.Error("failed to close connection", "error", closeErr)
    }
}()
```
</pattern>

## Interface Design (ISP Implementation)

<guideline type="interface_size">
**Small, focused interfaces (ISP):**
</guideline>

<pattern type="interface_segregation">
```go
// ✅ Good: Small, focused interfaces
type Reader interface {
    Read(ctx context.Context, id core.ID) (*Data, error)
}

type Writer interface {
    Write(ctx context.Context, data *Data) error
}

type Deleter interface {
    Delete(ctx context.Context, id core.ID) error
}

// Compose when needed
type Repository interface {
    Reader
    Writer
    Deleter
}

// ❌ Bad: Monolithic interface
type DataManager interface {
    Read(ctx context.Context, id core.ID) (*Data, error)
    Write(ctx context.Context, data *Data) error
    Delete(ctx context.Context, id core.ID) error
    Backup(ctx context.Context) error
    Restore(ctx context.Context) error
    Migrate(ctx context.Context) error
}
```
</pattern>

<pattern type="interface_definition">
```go
// Small, focused interfaces for specific domains
type Storage interface {
    SaveMCP(ctx context.Context, def *MCPDefinition) error
    LoadMCP(ctx context.Context, name string) (*MCPDefinition, error)
    Close() error
}
```
</pattern>

<best_practices type="interface_organization">
**Interface best practices:**
- Define interfaces in separate files when used across packages
- Keep interfaces small and focused on specific behavior
- Use interface composition for complex behavior
- Honor contracts consistently (LSP)
</best_practices>

## Constructor Patterns (DIP Implementation)

<requirement type="mandatory">
**MANDATORY for all services:**
</requirement>

<pattern type="dependency_injection">
```go
// ✅ Good: Depends on abstraction (DIP)
type WorkflowService struct {
    taskRepo TaskRepository // interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
