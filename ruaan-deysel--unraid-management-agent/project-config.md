---
trigger: always_on
description: > Single source of truth for all AI coding assistants working on this project.
---

# AGENTS.md — AI Agent Instructions

> Single source of truth for all AI coding assistants working on this project.
> Individual tool files (CLAUDE.md, GEMINI.md, copilot-instructions.md, .cursorrules) point here.

## Project Identity

| Key          | Value                                                                  |
| ------------ | ---------------------------------------------------------------------- |
| **Name**     | Unraid Management Agent                                                |
| **Language** | Go 1.26                                                                |
| **Target**   | Linux/amd64 (Unraid OS)                                                |
| **Type**     | Third-party community plugin (not official Unraid)                     |
| **Purpose**  | REST API + WebSocket + MCP interface for system monitoring and control |
| **Repo**     | `github.com/ruaan-deysel/unraid-management-agent`                      |

## Project Structure

```text
/
├── daemon/                     # Application source code
│   ├── cmd/                    # CLI entry points (boot command)
│   ├── constants/              # System paths, binary locations, collection intervals
│   ├── domain/                 # Core domain types (Context, Config)
│   ├── dto/                    # Data Transfer Objects (shared structs)
│   ├── lib/                    # Utilities: shell exec, parsing, validation
│   ├── logger/                 # Logging wrapper (lumberjack)
│   ├── platform/               # OS-resilience: source-health registry, capability detection, path/binary resolution
│   └── services/
│       ├── api/                # HTTP server, REST handlers, WebSocket hub
│       ├── agent/              # Embedded autonomous agent (LLM loop, tools, sessions)
│       ├── alerting/           # Alert engine, evaluation, dispatch
│       ├── collectors/         # Data collection goroutines
│       ├── controllers/        # Control operations (Docker, VM, Array, etc.)
│       ├── mcp/                # Model Context Protocol server for AI agents
│       ├── mqtt/               # MQTT client for Home Assistant integration
│       └── watchdog/           # Health probes, remediation, runner
├── docs/                       # Documentation (API, MCP, WebSocket, guides)
│   └── integrations/           # AI/automation guides: mcp, claude/, chatgpt/, mqtt, grafana
├── skills/                     # Agent Skill (Agent Skills standard) — unraid-management-agent/
├── .claude-plugin/             # Claude Code plugin marketplace manifest for the skill
├── meta/                       # Plugin metadata (XML, page files, scripts)
├── scripts/                    # Developer setup helpers
├── tests/                      # Integration tests
├── .github/
│   ├── instructions/           # Path-specific AI instructions (applyTo globs)
│   └── prompts/                # Reusable task prompts for common workflows
├── Makefile                    # Build automation
├── go.mod / go.sum             # Go dependencies
├── CHANGELOG.md                # Release notes (MUST be updated with every change)
├── VERSION                     # Current version (YYYY.MM.DD format)
└── CONTRIBUTING.md             # Contribution guidelines
```

## Architecture

### Event-Driven PubSub Design

```
Collectors → Event Bus (github.com/cskr/pubsub) → API Server Cache → REST Endpoints
                                                 ↓                   ↓
                                          WebSocket Hub        MCP Server → AI Agents
                                                 ↓
                                          Connected Clients
```

### Critical Initialization Order

In `daemon/services/orchestrator.go`:

1. API server creates subscriptions via `Hub.Sub()` **FIRST**
2. 100ms delay ensures subscriptions are ready
3. Then collectors start publishing via `Hub.Pub(data, "topic_name")`

**Never change this order** — collectors publishing before subscriptions causes lost events.

### Native API Integration

Prefer native Go libraries over shell commands:

| Component | Library                              | Purpose                 |
| --------- | ------------------------------------ | ----------------------- |
| Docker    | `github.com/moby/moby/client`        | Docker Engine SDK       |
| VMs       | `github.com/digitalocean/go-libvirt` | Native libvirt bindings |
| System    | Direct `/proc`, `/sys` access        | Kernel interfaces       |

### Data Flow Example

1. **System Collector** reads CPU/RAM from `/proc/meminfo`, `/proc/stat`
2. Publishes `dto.SystemInfo` to event bus topic `system_update`
3. **API Server** receives event, updates `systemCache`
4. **WebSocket Hub** receives event, broadcasts to all clients
5. **REST endpoint** `/api/v1/system` returns cached `systemCache` data

### Core Components

#### Domain Layer (`daemon/domain/`)

- `Context`: Application runtime context holding the PubSub hub and configuration
- `Config`: Configuration settings (version, port)

#### Data Transfer Objects (`daemon/dto/`)

All data structures shared between collectors, API, and WebSocket clients:

- `SystemInfo`, `ArrayStatus`, `DiskInfo`, `NetworkInfo`
- `ContainerInfo`, `VMInfo`, `UPSStatus`, `GPUMetrics`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruaan-deysel/unraid-management-agent](https://github.com/ruaan-deysel/unraid-management-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
