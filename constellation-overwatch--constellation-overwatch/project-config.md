---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This project uses [Task](https://taskfile.dev/) instead of Make for task automation.

### Development
```bash
# List all available tasks
task --list

# Development mode (with templ auto-rebuild and server)
task dev

# Run the server
go run ./cmd/microlith/main.go

# Build the binary
task build

# Run the built binary
task run

# Run with TUI dashboard
task tui

# Build + run with TUI
task build-tui

# Run tests
go test ./...

# Install dependencies
go mod download

# Tidy dependencies
go mod tidy

# Format code
go fmt ./...

# Run go vet
go vet ./...

# Generate templ templates
task templ-generate

# Watch templ files for changes
task templ-watch

# Clean templ cache and regenerate
task templ-clean

# Clean build artifacts and data
task clean

# Generate development certificates
task generate-certs

# Docker commands
task docker-build
task docker-run
task docker-stop
```

### Environment Setup
Create a `.env` file from the example:
```bash
cp .env.example .env
```

Default ports:
- API Server & Web UI: 8080
- NATS Server: 4222 (embedded)

## Architecture

### Overview
Constellation Overwatch is an industrial data fabric and C4 toolbelt for agentic drones, robots, sensors, and video streams. It's built as a monolithic "microlith" with embedded NATS JetStream for messaging.

**Module:** `github.com/Constellation-Overwatch/constellation-overwatch` (Go 1.25)

### Core Components

#### 1. API Layer (`/api`)
- **handlers/**: HTTP endpoint handlers (entities, organizations, telemetry, commands)
- **middleware/**: Authentication (API key, session, WebAuthn passkeys) and CORS
- **responses/**: Standard JSON response utilities (`SendSuccess`, `SendError`)
- **services/**: Business logic for entity, organization, auth, and session management

#### 2. Database Layer (`/db`)
- SQLite database with auto-initialization (`MaxOpenConns=1`)
- Schema defined in embedded `schema.sql`
- Service pattern with connection management

#### 3. Event System (`/pkg/services/workers`)
- **manager.go**: Orchestrates all workers with graceful lifecycle
- **entity.go**: Processes entity CRUD events
- **command.go**: Handles command distribution
- **telemetry.go**: Processes telemetry streams
- **event.go**: General event processing
- **base.go**: Defines `Worker` interface

#### 4. NATS Integration (`/pkg/services/embedded-nats`)
- Embedded NATS server with JetStream
- Pull-based subscriptions (not drain-based)
- Streams:
  - CONSTELLATION_ENTITIES: Entity lifecycle events
  - CONSTELLATION_COMMANDS: Command messages
  - CONSTELLATION_TELEMETRY: Telemetry data
  - CONSTELLATION_EVENTS: General events

#### 5. Web UI (`/pkg/services/web`)
- Templ for type-safe server-side HTML templates
- Datastar for dynamic HTML updates
- SSE for real-time entity/telemetry updates

#### 6. TUI Dashboard (`/pkg/tui`)
- Bubbletea-based terminal UI
- Custom zap `TUIHook` captures logs for display

#### 7. Metrics (`/pkg/metrics`)
- Prometheus metrics collectors
- Entity registry with in-memory cache synced to NATS KV store

### Key Patterns

#### Service Lifecycle
All major components implement the `Service` interface:
```go
type Service interface {
    Start(ctx context.Context) error
    Stop(ctx context.Context) error
    Name() string
    HealthCheck() error
}
```
Services register with the Service Manager, start in registration order, stop in reverse order with 5-second timeout.

#### Event-Driven Architecture
All state changes publish events to NATS subjects:
- `constellation.entities.{org_id}.created`
- `constellation.entities.{org_id}.updated`
- `constellation.entities.{org_id}.deleted`
- `constellation.telemetry.{org_id}.{entity_id}`
- `constellation.commands.{org_id}.{entity_id}`

#### Authentication
Multiple auth mechanisms:
- **API Key**: Bearer token in `Authorization` header (for API clients)
- **Session**: In-memory session store with tokens (for web UI)
- **WebAuthn**: Passkey-based authentication (for web UI)
- **Bootstrap**: Creates admin user on first run with invite token

#### Standard Response Format
```go
type Response struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   *Error      `json:"error,omitempty"`
}
```

#### Database Auto-Initialization
The database service automatically:
- Creates the SQLite file if missing
- Initializes schema from embedded `schema.sql`
- Verifies schema on startup

### Key Dependencies
- `go-chi/chi/v5`: HTTP router
- `a-h/templ`: Type-safe templates
- `nats-io/nats-server/v2`: Embedded NATS
- `charmbracelet/bubbletea`: Terminal UI
- `go.uber.org/zap`: Structured logging
- `modernc.org/sqlite`: Pure-Go SQLite
- `starfederation/datastar-go`: Dynamic HTML
- `prometheus/client_golang`: Metrics
- `go-webauthn/webauthn`: Passkey auth

### Entity Types
Defined in `prd/design/CONSTELLATION_TAK_ONTOLOGY.md` and `pkg/ontology/`:
- `aircraft_multirotor`: Drones/UAVs
- `ground_vehicle`: Ground robots/vehicles
- `sensor_fixed`: Stationary sensors
- `control_station`: Command centers
- And many more...

### Important Files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Constellation-Overwatch/constellation-overwatch](https://github.com/Constellation-Overwatch/constellation-overwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
