---
trigger: always_on
description: **Go Version:** 1.22+
---

# GoOps Forge Skill

**Version:** 1.0
**Go Version:** 1.22+
**Target Environment:** Linux (production deployments)

---

## 1. Skill Identity

### Name
GoOps Forge Skill

### Purpose
Teaches AI coding agents how to design, scaffold, review, and improve production-grade Go projects for Linux environments. Covers APIs, CLIs, daemons, workers, schedulers, webhooks, gRPC services, and AI agent runtimes.

### Target Projects
- REST/gRPC APIs
- CLI tools
- Worker daemons and queue consumers
- Scheduled job services
- Webhook receivers
- Linux system agents
- MCP backends and agent runtimes
- DevOps and internal tooling
- Backend-for-frontend services

### Non-Goals
- Toy scripts or learning exercises
- Windows-specific GUI applications
- Mobile applications
- Projects explicitly marked "not for production"
- Experiments that will be discarded

---

## 2. When to Activate This Skill

Activate when the user asks for anything matching these patterns:

| User Request | Skill Action |
|-------------|-------------|
| "build a Go CLI" | Use `templates/cli/`; follow `recipes/create-new-cli.md` |
| "build a Go API server" | Use `templates/rest-api/`; follow `recipes/create-new-api-service.md` |
| "build a Go worker/consumer" | Use `templates/worker-daemon/`; follow `recipes/create-new-worker.md` |
| "build a Go daemon for Linux" | Use `templates/linux-agent/`; add systemd via `recipes/add-docker-systemd.md` |
| "build a scheduler/cron in Go" | Use `templates/scheduler/` |
| "build a webhook receiver" | Use `templates/webhook-service/`; follow `recipes/create-new-webhook-service.md` |
| "build a gRPC service" | Use `templates/grpc-service/` |
| "build an MCP backend in Go" | Use `templates/rest-api/` + observability via `recipes/add-observability.md` |
| "build an agent runtime in Go" | Use `templates/linux-agent/` + graceful shutdown + observability |
| "improve my Go repo" | Run `scripts/audit_go_service.sh`; follow `recipes/improve-existing-repo.md` |
| "make my Go service production-ready" | Follow `recipes/prepare-for-production.md` |
| "add tests to my Go project" | Follow `recipes/add-tests.md` |
| "review my Go code" | Use `review/review-prompt.md` with `go-service-scorecard.md` |
| "walk through building X" | Use `examples/` walkthroughs (e.g., `examples/01-build-rest-api.md`) |

**Deactivation:** Do not activate if the user explicitly says "don't use this skill," "it's just a quick hack," or "learning Go." If unsure, activate — the skill cost is low.

---

## 3. Agent Operating Rules

Every agent working on a Go project must follow these rules without exception.

### 3.1 Inspect Before Changing

Before writing any code in an existing repository:

```bash
# 1. Check git state
git status --short
git branch --show-current

# 2. Find the module path
head -3 go.mod  # Module name is in line 2

# 3. Check Go version
go version

# 4. Check project structure
find . -maxdepth 3 -type f -name "*.go" | head -20
find . -maxdepth 2 -type d | sort

# 5. Check for existing tests
find . -name "*_test.go" | wc -l

# 6. Check Docker/Compose files
ls -la Dockerfile* docker-compose* 2>/dev/null || true
```

### 3.2 Detect Module Path

Always use the **correct module path** from `go.mod`. Never guess.

```
# From go.mod line 2:
module github.com/myorg/myproject

# Correct import:
import "github.com/myorg/myproject/internal/service"
```

### 3.3 Detect Go Version

Check `go.mod` for the minimum Go version:

```
go 1.22
```

Use only features available in that version. Default to Go 1.22+ features (slog, slices.Concat, maps etc.).

### 3.4 Project Structure Rules

**AVOID** single-file production services. A production service requires:

```
myproject/
├── cmd/           # ONE SUBDIR PER BINARY (never main.go in root)
│   └── myservice/
│       └── main.go
├── internal/      # Private application code (not importable)
│   ├── config/
│   ├── domain/
│   ├── service/
│   ├── repository/
│   ├── handler/
│   └── infrastructure/
├── pkg/           # Only for truly public libraries
├── api/           # OpenAPI specs, proto files
├── migrations/    # SQL migrations (goose)
├── configs/      # Config templates (.tmpl)
├── scripts/       # Build/deployment scripts
├── Dockerfile
├── docker-compose.yml
├── Makefile
└── go.mod
```

### 3.5 Context.Context Rules

Every function doing I/O, waiting, or anything that can timeout MUST accept `context.Context`:

```go
// CORRECT
func (s *UserService) GetUser(ctx context.Context, id int64) (*User, error)

// WRONG - missing context
func (s *UserService) GetUser(id int64) (*User, error)

// NEVER store context in struct fields
type BadService struct {
    ctx context.Context  // WRONG
    db  *sql.DB
}
```

Context rules:
- `context.Context` is always the **first parameter**
- Never pass `context.Background()` to deep call chains — let the caller provide it
- Always wrap with timeout for external calls: `context.WithTimeout(ctx, 5*time.Second)`
- Check `ctx.Done()` in loops

### 3.6 Graceful Shutdown Rules

Every long-running service MUST handle SIGTERM and SIGINT:

```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: mux}

    go func() {
        if err := srv.ListenAndServe(); err != nil {
            slog.Error("server error", "error", err)
        }
    }()

    // Wait for signal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arvuno/GO-ops-Forge-SKill](https://github.com/Arvuno/GO-ops-Forge-SKill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
