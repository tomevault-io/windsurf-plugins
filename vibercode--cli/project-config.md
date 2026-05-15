---
trigger: always_on
description: ViberCode CLI es una herramienta de línea de comandos para generar APIs Go con arquitectura limpia, incluyendo operaciones CRUD completas, integración con bases de datos, y un sistema de chat AI interactivo.
---

# ViberCode CLI

## Información General

ViberCode CLI es una herramienta de línea de comandos para generar APIs Go con arquitectura limpia, incluyendo operaciones CRUD completas, integración con bases de datos, y un sistema de chat AI interactivo.

## Nuevas Características

### 🔌 Servidor MCP (Model Context Protocol)

El nuevo servidor MCP permite que agentes de IA interactúen directamente con ViberCode para:

- **Edición de componentes en vivo**: Actualizar propiedades, posición y tamaño de componentes en tiempo real
- **Chat integrado**: Enviar mensajes al asistente Viber AI desde agentes externos
- **Generación de código**: Crear APIs Go completas usando agentes IA
- **Gestión de estado**: Obtener y actualizar el estado de la vista y componentes

#### Uso Rápido

```bash
# Iniciar servidor MCP
vibercode mcp

# Probar el servidor
./test-mcp-server.sh
```

#### Configuración para Claude Desktop

Agregar al archivo `.mcp.json`:

```json
{
  "mcpServers": {
    "vibercode": {
      "name": "ViberCode MCP Server",
      "description": "ViberCode CLI integration for live component editing",
      "command": "vibercode",
      "args": ["mcp"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
        "VIBE_DEBUG": "true"
      }
    }
  }
}
```

#### Herramientas Disponibles

- `vibe_start`: Iniciar modo vibe con chat AI y preview
- `component_update`: Actualizar componentes en tiempo real
- `view_state_get`: Obtener estado actual de la vista
- `chat_send`: Enviar mensaje al asistente Viber AI
- `generate_code`: Generar código Go API
- `project_status`: Estado del proyecto y servidores

## Arquitectura

### CLI Structure

```
vibercode-cli-go/
├── main.go                      # Application entry point
├── cmd/                         # Cobra CLI commands
│   ├── root.go                 # Root command definition
│   └── generate.go             # Generate subcommands
├── internal/
│   ├── generator/              # Code generation logic
│   │   ├── api.go             # API project generator
│   │   └── resource.go        # Resource CRUD generator
│   ├── models/                 # Data structures
│   │   └── field.go           # Field types and resource models
│   └── templates/              # Go template strings
│       ├── model.go           # Model template
│       ├── handler.go         # HTTP handler template
│       ├── service.go         # Business logic template
│       └── repository.go      # Data access template
├── go.mod                      # Go module definition
└── README.md                   # Documentation
```

### Generated Project Architecture

The CLI generates Go projects following clean architecture principles:

```
generated-project/
├── cmd/server/main.go          # Application entry point
├── internal/
│   ├── handlers/               # HTTP layer (Gin framework)
│   ├── services/               # Business logic layer
│   ├── repositories/           # Data access layer
│   ├── models/                 # Domain models and DTOs
│   └── middleware/             # HTTP middleware
├── pkg/
│   ├── database/               # Database connection utilities
│   ├── config/                 # Configuration management
│   └── utils/                  # Shared utilities
└── go.mod                      # Go module
```

### Integración MCP

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   AI Agent      │    │   MCP Server    │    │   ViberCode     │
│   (Claude)      │◄──►│   (JSON-RPC)    │◄──►│   Services      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   WebSocket     │
                       │   + HTTP API    │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  React Editor   │
                       │  + Live Preview │
                       └─────────────────┘
```

## Comandos Disponibles

### CLI Commands

- `vibercode generate api` - Generate a complete Go API project with clean architecture
- `vibercode generate resource` - Generate CRUD resources with models, handlers, services, and repositories
- `vibercode --help` - Show help information
- `vibercode generate --help` - Show help for generate commands

### Development Commands

- `go build -o vibercode main.go` - Build the CLI binary
- `go mod tidy` - Clean up dependencies
- `go test ./...` - Run tests
- `go run main.go` - Run CLI during development

### `vibercode mcp`

Inicia el servidor MCP para integración con agentes IA.

```bash
vibercode mcp
```

**Características:**

- Protocolo MCP 2024-11-05 compatible
- Comunicación JSON-RPC via stdin/stdout
- Herramientas bien definidas con validación
- Integración con WebSocket y HTTP API

## Key Features

### Field Type System

The CLI supports a comprehensive field type system defined in `internal/models/field.go`:

- **Basic Types**: `string`, `text`, `number`, `float`, `boolean`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibercode/cli](https://github.com/vibercode/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
