---
trigger: always_on
description: Short, actionable guidance so an AI agent can be productive immediately in this repo.
---

# Copilot / AI Agent Instructions for FileCodeBox

Short, actionable guidance so an AI agent can be productive immediately in this repo.

1) Big picture
- Layered architecture: `routes -> handlers -> services -> repository -> database/storage`.
- Key directories: `internal/routes/`, `internal/handlers/`, `internal/services/`, `internal/repository/`, `internal/storage/`, `internal/config/`, `internal/mcp/`, `docs/`.
- Main entry: `main.go` initializes `ConfigManager`, `StorageManager`, and starts the HTTP server via `routes.CreateAndStartServer()`; DB initialization is deferred and can be triggered via `/setup/initialize`.

2) Typical data & control flow
- HTTP requests handled by `routes` -> call `handlers` -> call `services` -> use `repository` -> talk to DB/storage.
- Storage is abstracted by `storage.NewStorageManager(manager)` and concrete implementations under `internal/storage/` (local, s3, webdav, onedrive).
- MCP subsystem lives under `internal/mcp/` and is conditionally started when `manager.MCP.EnableMCPServer == 1`.

3) Configuration & runtime
- Config is centralized in `internal/config/manager.go` (use `config.InitManager()` to obtain `ConfigManager`).
- Environment variables override config; `manager.SetDB(db)` is used to inject DB connection after initialization.
- Common env vars: `PORT`, `DATA_PATH`, `ENABLE_MCP_SERVER`.

4) Versioning & release patterns
- Project version stored in the top-level `VERSION` file and echoed in `internal/models/service/system.go` (`Version` variable) and swagger docs under `docs/`.
- Releases are created by updating those files and tagging the commit (e.g., `v1.8.2`). Avoid editing `go.sum` manually.

5) Build, test, and release commands
- Build: `make build` or `go build ./...`.
- Tests: `make test` or `go test ./...` (many packages have no tests; run targeted packages if needed).
- Docker: `./scripts/build-docker.sh` and `docker-compose.yml` present.

6) Project-specific conventions
- Handler constructors follow: `NewXxxHandler(service *services.XxxService, config *config.ConfigManager) *XxxHandler`.
- Services are created with dependency injection in `routes` during server start: e.g., `services.NewUserService(daoManager, manager)`.
- Repositories live under `internal/repository/` and expose `RepositoryManager` for DAOs (use `dmgr *repository.RepositoryManager`).
- Error responses use helper functions in `internal/common/` (`common.SuccessResponse`, `common.ErrorResponse`, `common.BadRequestResponse`).

7) Integration points to inspect when editing code
- `internal/config/manager.go` — config lifecycle, hot reload hooks.
- `internal/routes/setup.go` — server and route wiring, DI order.
- `internal/mcp/manager.go` — MCP lifecycle and tools registration.
- `internal/storage/*` — adding new storage backends: implement `storage.StorageInterface` and register in `storage.NewStorageManager`.

8) Useful file examples to copy patterns from
- `internal/services/*` shows DI, error wrapping and transaction handling (e.g., `internal/services/admin/*`).
- `internal/routes/*` shows route grouping and middleware usage, look at `internal/routes/admin.go` for admin auth patterns.

9) CSS / Frontend notes
- Frontend themes under `themes/2025/`. Admin UI assets are served via protected routes — modifying admin CSS may require rebuilding or restarting server to pick static changes when not using hot reload.

10) Safety & version control
- Do not alter `go.sum` manually.
- When creating tags, prefer not to overwrite remote tags; create a new semver or `-local.<sha>` tag if necessary.

If any section is unclear or you want examples added (e.g., sample PR description, changelog generation command), tell me which part to expand. After your feedback I'll iterate.
# FileCodeBox AI Coding Agent Instructions

## Project Overview

FileCodeBox 是一个高性能的文件快传系统的 Go 实现，基于现代化的分层架构设计。该项目提供文件/文本分享、分片上传、用户系统、多存储后端支持，以及独特的 Model Context Protocol (MCP) 服务器集成。

## Architecture Patterns

### Layered Architecture
遵循严格的分层架构：`routes → handlers → services → repository → database/storage`

- **Routes** (`internal/routes/`): 按功能模块化路由 - `base.go`, `share.go`, `user.go`, `chunk.go`, `admin.go`
- **Handlers** (`internal/handlers/`): HTTP 处理器，负责请求解析和响应
- **Services** (`internal/services/`): 业务逻辑层，处理核心业务
- **Repository** (`internal/repository/`): 数据访问层，提供统一的 DAO 接口
- **Storage** (`internal/storage/`): 存储抽象层，支持 local/s3/webdav/onedrive

### Configuration Management
采用分层配置管理模式 (`internal/config/`):
```go
// 通过 ConfigManager 统一管理所有配置
manager := config.InitManager()
manager.SetDB(db) // 注入数据库连接（配置读取现在以 config.yaml 和 环境变量为准）
```

配置分为多个模块：`BaseConfig`, `DatabaseConfig`, `StorageConfig`, `UserSystemConfig`, `MCPConfig`

支持环境变量优先级覆盖、数据库持久化存储、热重载机制：
- **环境变量优先级**：PORT、DATA_PATH 等关键配置始终优先使用环境变量
- **数据库持久化**：配置自动保存到 key_value 表，支持动态更新
- **热重载机制**：通过 ReloadConfig() 方法实现运行时配置更新
- **配置验证**：每个配置模块都有独立的验证方法
- **类型安全配置**：使用结构体 Clone() 方法和直接字段访问，避免 map 转换开销

### Route Architecture
完全模块化的路由系统 (`internal/routes/`):
```go
// 自动化服务器创建和启动
srv, err := routes.CreateAndStartServer(manager, daoManager, storageManager)

// 分层路由管理
SetupAllRoutesWithDependencies(router, manager, daoManager, storageManager)
```

路由按功能分组：`base.go`, `share.go`, `user.go`, `chunk.go`, `admin.go`
自动依赖注入：服务→处理器→路由的完整初始化链


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zy84338719/FileCodeBox](https://github.com/zy84338719/FileCodeBox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
