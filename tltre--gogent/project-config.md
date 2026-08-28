---
trigger: always_on
description: - **Module**: `github.com/tltre/gogent`
---

# AGENTS.md — Gogent

## Project at a Glance

- **Module**: `github.com/tltre/gogent`
- **Go**: 1.25.5
- **Version**: v0.15.0
- **Entrypoint**: `cmd/gogent/main.go` — cobra-based CLI (`gogent run/status/doctor/...`)
- **Binary**: `gogent`
- **Deps**: `gopkg.in/yaml.v3` (direct), `github.com/mark3labs/mcp-go` (indirect), `go.uber.org/zap` (indirect), `github.com/spf13/cobra` (direct)
- **CI**: GitHub Actions (`.github/workflows/ci.yml`) — go build + go vet + go test
- **Test helper**: `cmd/daemon/` — stdio daemon binary for end-to-end smoke testing

## Development Commands

```bash
go build ./...                              # build all packages
go vet ./...                                # static analysis
go test ./tests/native/ -v -timeout 30s     # native mock tests (13)
go test ./tests/stdio/ -v -timeout 60s      # stdio round-trip tests (13)
go test ./tests/http/ -v -timeout 30s       # http round-trip tests (9)
go test ./tests/e2e/ -v -timeout 60s        # full integration test (1)
go test ./tests/cli/ -v -timeout 30s        # CLI framework tests (19)
```

## Architecture

### Component system

Every subsystem is a `component.Component` (interface in `pkg/component/component.go`):

```go
type Component interface {
    GetName() string
    GetType() ComponentType
    Initialize(ctx context.Context, deps *Registry) error
    Start(ctx context.Context) error
    Stop(ctx context.Context) error
    Dependencies() map[string]DependencySpec
}
```

A **Registry** (`pkg/component/registry.go`) holds components and does topological-sort initialization. Every component wrapping struct implements `Component` and delegates to a "business interface" (e.g., `IMemory`, `IProvider`, `IAgentCore`).

10 component types: `ComponentChannel`, `ComponentAgentCore`, `ComponentProvider`, `ComponentTool`, `ComponentHook`, `ComponentEventBus`, `ComponentContextManager`, `ComponentMemory`, `ComponentSandbox`, `ComponentLogger`.

### BasicComponent (v0.3.1)

All component wrappers embed `component.BasicComponent` to share common fields:

```go
type BasicComponent struct {
    name string
    reg  *Registry
}

func (b *BasicComponent) GetName() string
func (b *BasicComponent) Registry() *Registry
func (b *BasicComponent) SetRegistry(r *Registry)
```

Embedding eliminates per-component `name`/`reg` fields and `GetName()` boilerplate. Ten component types: `ComponentChannel`, `ComponentAgentCore`, `ComponentProvider`, `ComponentTool`, `ComponentHook`, `ComponentEventBus`, `ComponentContextManager`, `ComponentMemory`, `ComponentSandbox`, `ComponentLogger`.

### Interface Layer (v0.4.0)

The **interface layer** is NOT a Component — it is a top-level abstraction that drives the application's interaction loop. It starts after all backend Components are started and blocks until user exit.

```go
// pkg/iface/iface.go
type Interface interface {
    Run(ctx context.Context, reg *component.Registry) error
}
```

**Key design decisions:**
- Not registered in Registry; not a `component.Component`
- Not a ComponentType; YAML driver dispatch does not apply
- `Run()` is blocking — replaces `<-ctx.Done()` in `App.Run()`
- Interface types are **mutually exclusive** per application instance

**Three implementations** (phased rollout):

| Type | YAML key | Description | Version |
|------|----------|-------------|---------|
| CLI | `type: cli` | cobra-based extensible command tree | v0.4.2 |
| TUI | `type: tui` | bubbletea interactive terminal | deferred |
| HTTP | `type: http` | embedded web server | deferred |

### v0.4.x Roadmap

```
v0.4.1 — 框架搭建
    ├── pkg/iface/iface.go          Interface 接口
    ├── pkg/app/config.go           追加 InterfaceConfig
    ├── pkg/app/builder.go          追加 buildInterface() + WithInterface()
    └── pkg/app/app.go              追加 iface 字段 + Run() 改造（iface != nil 时调 Run，else <-ctx.Done）

v0.4.2 — CLI 命令体系
    ├── pkg/iface/cli/              CLI 子包（命令树 + 注册/卸载 API）
    │   ├── cli.go                 DefaultCLI 结构体 + Register/Unregister/Run
    │   ├── command.go             CommandEntry + CommandBuilder + RegisterByPath
    │   ├── cmd_chat.go            默认命令: chat (交互式 REPL)
    │   ├── cmd_run.go             默认命令: run -p "..." (非交互单次调用)
    │   ├── cmd_version.go         默认命令: version
    │   ├── cmd_config.go          预制命令: config validate
    │   ├── cmd_tools.go           预制命令: tools (TODO)
    │   ├── cmd_providers.go       预制命令: providers (TODO)
    │   ├── cmd_sessions.go        预制命令: sessions (TODO)
    │   ├── cmd_memory.go          预制命令: memory (TODO)
    │   └── cmd_sandbox.go         预制命令: sandbox (TODO)
    └── pkg/app/builder.go         新增 WithCLICommand() BuildOption

v0.4.3 — TUI 实现
    ├── pkg/iface/tui.go            DefaultTUI (bubbletea)
    └── 依赖: github.com/charmbracelet/bubbletea + bubbles
    └── ⚠ deferred — 条件不成熟，待 Provider/AgentCore 核心流程稳定后再实现

v0.4.4 — HTTP 默认实现
    ├── pkg/iface/http.go           DefaultHTTP (net/http)
    └── 默认: iface.type 未配置时回退为 http
     └── ⚠ deferred — 条件不成熟，待 Provider/AgentCore 核心流程稳定后再实现
```

### Management Channel (v0.5.0)

The **management channel** is a framework-internal HTTP server that runs alongside the agent application after all backend Components are started. It exposes a REST API for CLI/TUI/HTTP management tools to observe and interact with the running agent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tltre/gogent](https://github.com/tltre/gogent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
