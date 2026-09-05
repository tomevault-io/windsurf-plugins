---
trigger: always_on
description: This document provides context for AI agents and contributors working on the OpenHands Kubernetes Remote Runtime project.
---

# Agent Context for OpenHands Kubernetes Remote Runtime

This document provides context for AI agents and contributors working on the OpenHands Kubernetes Remote Runtime project.

## Project Overview

The OpenHands Kubernetes Remote Runtime is a Go-based service that provisions sandbox pods in Kubernetes for OpenHands agent sessions. It implements the OpenHands Remote Runtime API contract and provides subdomain-based routing for agent servers, VSCode, and worker processes.

## Architecture

### Core Components

1. **API Server** (`pkg/api/handler.go`)
   - HTTP REST API with 11 endpoints
   - Authentication via X-API-Key header
   - Request validation and error handling
   - Structured JSON responses

2. **Kubernetes Client** (`pkg/k8s/client.go`)
   - Pod provisioning with resource limits
   - Service creation for port exposure
   - Ingress creation with subdomain routing
   - Pod status monitoring and health checks
   - **Note**: Currently has 0% test coverage due to lack of mocking infrastructure

3. **State Management** (`pkg/state/state.go`)
   - In-memory runtime state tracking
   - Thread-safe operations with mutex locks
   - Efficient lookups by runtime ID and session ID
   - Activity timestamp tracking for idle timeout
   - **Test Coverage**: 100%

4. **Idle Sandbox Reaper** (`pkg/reaper/reaper.go`)
   - Background goroutine for automatic cleanup
   - Configurable idle timeout (default: 12 hours)
   - Periodic checks (default: every 15 minutes)
   - Graceful cleanup of idle sandboxes
   - **Test Coverage**: ~90%

5. **Configuration** (`pkg/config/config.go`)
   - Environment-based configuration
   - Sensible defaults for all settings
   - **Test Coverage**: 100%

6. **Type Definitions** (`pkg/types/types.go`)
   - Request/response models
   - Status enumerations
   - Shared data structures

## API Contract

The service implements the exact API contract expected by OpenHands Remote Runtime:

### Key Endpoints

- `POST /start` - Start new runtime sandbox
- `POST /stop` - Stop running runtime
- `POST /pause` - Pause runtime (delete pod, keep state)
- `POST /resume` - Resume paused runtime (recreate pod)
- `GET /list` - List all runtimes
- `GET /runtime/{runtime_id}` - Get runtime details
- `GET /sessions/{session_id}` - Get session by ID
- `GET /sessions/batch` - Batch query sessions
- `GET /registry_prefix` - Get container registry prefix
- `GET /image_exists` - Check if image exists
- `GET /health` - Health check endpoint (no auth required)
- `GET /liveness` - Liveness probe endpoint (no auth required)
- `GET /readiness` - Readiness probe endpoint (no auth required)

### Response Format

All responses follow this structure:

```go
type RuntimeResponse struct {
    RuntimeID      string            `json:"runtime_id"`
    SessionID      string            `json:"session_id"`
    URL            string            `json:"url"`
    SessionAPIKey  string            `json:"session_api_key"`
    Status         RuntimeStatus     `json:"status"`
    PodStatus      PodStatus         `json:"pod_status"`
    WorkHosts      map[string]int    `json:"work_hosts"`
    RestartCount   int               `json:"restart_count,omitempty"`
    RestartReasons []string          `json:"restart_reasons,omitempty"`
}
```

## Subdomain Routing

Each sandbox gets unique subdomains:
- `{session-id}.sandbox.example.com` → Agent server (port 60000)
- `vscode-{session-id}.sandbox.example.com` → VSCode (port 60001)
- `work-1-{session-id}.sandbox.example.com` → Worker 1 (port 12000)
- `work-2-{session-id}.sandbox.example.com` → Worker 2 (port 12001)

## Pod Specification

Each sandbox pod includes:
- Agent server container with OpenHands runtime image
- 4 exposed ports (agent, vscode, worker1, worker2)
- Environment variables for session API key, webhooks, CORS
- Resource requests and limits (configurable via resource_factor)
- Readiness probe on /alive endpoint
- Support for custom runtime classes (sysbox-runc, gvisor)
- Optional imagePullSecrets when `IMAGE_PULL_SECRETS` is set (for private registries)
- Optional CA cert mount when `CA_CERT_SECRET_NAME` is set (for corporate/proxy CAs); cert is mounted at `/usr/local/share/ca-certificates/additional-ca.crt` and merged via `update-ca-certificates` at runtime startup

## Development Guidelines

### Adding New Features

1. **Follow existing patterns**: Look at similar functionality in the codebase
2. **Add tests**: Maintain or improve test coverage (current: 26.5%)
3. **Update documentation**: Keep README.md, CONTRIBUTING.md, and this file in sync
4. **Run pre-commit checks**: `make pre-commit` before committing

### Testing Strategy

- **Unit tests**: Focus on business logic and state management
- **Table-driven tests**: Preferred pattern for multiple test cases
- **Mocking**: Currently needed for Kubernetes client testing (future work)
- **Coverage threshold**: Minimum 25% (enforced by CI)

### Current Test Coverage

| Package | Coverage | Priority |
|---------|----------|----------|
| pkg/config | 100% | ✅ Complete |
| pkg/state | 100% | ✅ Complete |
| pkg/reaper | ~90% | ✅ Complete |
| pkg/api | ~28% | ⚠️ Needs improvement |
| pkg/types | N/A | N/A |
| pkg/k8s | 0% | ⚠️ High priority - needs mocking |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zparnold/openhands-kubernetes-remote-runtime](https://github.com/zparnold/openhands-kubernetes-remote-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
