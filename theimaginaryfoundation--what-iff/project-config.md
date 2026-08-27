---
trigger: always_on
description: The API server is a RESTful service built with Go, using the Gorilla Mux router for routing and Ent for database operations. The server follows a modular design with clear separation of concerns and dependency injection patterns.
---

# API Server Design and Structure

## Overview

The API server is a RESTful service built with Go, using the Gorilla Mux router for routing and Ent for database operations. The server follows a modular design with clear separation of concerns and dependency injection patterns.

## Server Initialization

The API server is initialized in `cmd/api-server/main.go`. It follows these steps:

1. Load environment variables
2. Initialize logger
3. Connect to database
4. Create and configure server
5. Start server and handle graceful shutdown

```go
func main() {
    // Load environment variables
    if err := godotenv.Load(envFile); err != nil {
        log.Printf("Warning: Error loading .env file: %v", err)
    }

    // Initialize logger
    logger, err := zap.NewProduction()
    if err != nil {
        log.Fatalf("Failed to initialize logger: %v", err)
    }
    defer logger.Sync()

    // Initialize database connection
    client, err := database.NewClient(logger)
    if err != nil {
        logger.Fatal("Failed to connect to database", zap.Error(err))
    }
    defer client.Close()

    // Create server config
    config := server.NewConfig()

    // Create and configure server
    srv := server.NewServer(config, logger, client)

    // Run server in a goroutine
    go func() {
        if err := srv.Start(); err != nil {
            logger.Fatal("Server failed", zap.Error(err))
        }
    }()

    // Handle graceful shutdown
    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt, syscall.SIGTERM)
    sig := <-c
    
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*15)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        logger.Fatal("Server forced to shutdown", zap.Error(err))
    }
}
```

## Server Configuration

Server configuration is managed through the `Config` struct in `internal/server/config.go`. It loads settings from environment variables with sensible defaults.

```go
type Config struct {
    Host          string
    Port          string
    ReadTimeout   time.Duration
    WriteTimeout  time.Duration
    IdleTimeout   time.Duration
    RunMigrations bool
}

func NewConfig() *Config {
    port := os.Getenv("SERVER_PORT")
    if port == "" {
        port = "8080"
    }

    host := os.Getenv("SERVER_HOST")
    if host == "" {
        host = "localhost"
    }

    return &Config{
        Host:          host,
        Port:          port,
        ReadTimeout:   15 * time.Second,
        WriteTimeout:  15 * time.Second,
        IdleTimeout:   60 * time.Second,
        RunMigrations: false,
    }
}
```

## Server Structure

The server is structured around a central `Server` struct in `internal/server/server.go` which encapsulates:

- Configuration
- Logger
- Router
- HTTP server
- Database client

```go
type Server struct {
    config *Config
    logger *zap.Logger
    router *mux.Router
    server *http.Server
    db     *ent.Client
}

func NewServer(config *Config, logger *zap.Logger, db *ent.Client) *Server {
    s := &Server{
        config: config,
        logger: logger,
        router: mux.NewRouter(),
        db:     db,
    }

    s.setupMiddleware()
    s.setupRoutes()

    s.server = &http.Server{
        Addr:         fmt.Sprintf("%s:%s", config.Host, config.Port),
        Handler:      s.router,
        WriteTimeout: config.WriteTimeout,
        ReadTimeout:  config.ReadTimeout,
        IdleTimeout:  config.IdleTimeout,
    }

    return s
}
```

## Provider Pattern

We use the provider pattern to decouple handlers from data access implementations. Our approach has evolved to use composite provider interfaces for handlers that need access to multiple entity types.

### Entity-Specific Provider Interfaces

Each entity has its own provider interface in the `internal/providers/` directory:

```go
// ContentBriefProvider defines the interface for content brief data operations
type ContentBriefProvider interface {
    CreateContentBrief(ctx context.Context, userID uuid.UUID, contentBrief models.ContentBrief) (*models.ContentBrief, error)
    ListContentBriefs(ctx context.Context, userID uuid.UUID, pageNum, pageSize int, filters models.ContentBriefFilters) (*models.PaginatedResponse, error)
    GetContentBrief(ctx context.Context, userID, id uuid.UUID) (*models.ContentBrief, error)
    UpdateContentBrief(ctx context.Context, userID uuid.UUID, contentBrief models.ContentBrief) (*models.ContentBrief, error)
    DeleteContentBrief(ctx context.Context, userID, id uuid.UUID) error
}
```

### Composite Provider Interfaces

For handlers that need access to multiple entity types, we define composite interfaces that extend multiple entity provider interfaces:

```go
// ProjectBriefQuestionIdeaJobProvider combines multiple provider interfaces
type ProjectBriefQuestionIdeaJobProvider interface {
    providers.ProjectProvider
    providers.ContentIdeaProvider
    providers.ContentBriefProvider
    providers.InterviewQuestionProvider
    providers.JobProvider
}
```

This approach simplifies handler constructors and avoids large parameter lists.

### Generator Interfaces


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theimaginaryfoundation/what-iff](https://github.com/theimaginaryfoundation/what-iff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
