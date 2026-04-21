---
trigger: always_on
description: 后端 Go Modules Interfaces 层编码规范 - 接口层（HTTP/gRPC/事件）
---


# Modules Interfaces 层编码规范

本规则说明如何组织与编写 **modules/<module>/interfaces/** 下的代码，适用于采用分层架构的 Go 后端项目。接口层只依赖 Application 层，负责 DTO 转换与协议适配（HTTP、gRPC、事件等），不包含业务逻辑。

## 文件组织

```
modules/<module>/
└── interfaces/
    ├── http/                # HTTP 处理器
    │   ├── handler/         # 请求处理器（按实体）
    │   │   └── <entity>.go
    │   ├── dto/             # 数据传输对象
    │   │   ├── reqdto/      # 请求 DTO
    │   │   │   └── <entity>.go
    │   │   └── respdto/     # 响应 DTO
    │   │       └── <entity>.go
    │   ├── router.go        # 路由定义
    │   ├── server.go        # HTTP 服务器设置
    │   └── registry.go      # 处理器注册表
    ├── grpc/                # gRPC 处理器
    │   ├── handler/         # gRPC 处理器（按服务方法）
    │   │   └── <entity>.go
    │   ├── mapper/          # gRPC 映射器（Domain ↔ Protobuf）
    │   │   └── <entity>_mapper.go
    │   └── server.go        # gRPC 服务器设置
    └── pipeline/            # 事件处理器
        ├── handler/         # 事件处理器（按事件类型）
        │   └── <module>.go
        ├── registry.go      # 事件处理器注册表
        ├── router.go        # 事件路由器
        └── server.go        # 事件总线服务器
```

**示例**：
- `interfaces/http/handler/user.go` - User 的 HTTP 处理器
- `interfaces/grpc/handler/user.go` - User 的 gRPC 处理器
- 根据业务实体创建对应的处理器

## HTTP Handler（HTTP 处理器）

```go
// ✅ GOOD - HTTP 处理器标准结构
package handler

import (
    <entity>App "<project>/modules/<module>/application/<entity>"
    <entity>AppCommands "<project>/modules/<module>/application/<entity>/commands"
    "<project>/modules/<module>/interfaces/http/dto/reqdto"
    "<project>/modules/<module>/interfaces/http/dto/respdto"
    "<project>/pkgs/errx"
    "<project>/pkgs/fiberx"
    "<project>/pkgs/httpx"
    "github.com/gofiber/fiber/v3"
)

type <Entity>Handler struct {
    appSvc *<entity>App.Service
}

func New<Entity>Handler(appSvc *<entity>App.Service) *<Entity>Handler {
    return &<Entity>Handler{
        appSvc: appSvc,
    }
}

// Create 创建 <Entity>
func (h *<Entity>Handler) Create(c fiber.Ctx) error {
    var req reqdto.<Entity>CreateRequestDTO
    if err := c.Bind().Body(&req); err != nil {
        return errx.ErrInvalidBody.WithCause(err)
    }

    cmd := req.ToCreateCmd()
    ctx := c.Context()
    <entity>ID, err := h.appSvc.Create<Entity>(ctx, cmd)
    if err != nil {
        return err
    }

    <entity>View, err := h.appSvc.Get<Entity>(ctx, <entity>ID)
    if err != nil {
        return err
    }

    return fiberx.Created(c, "<Entity> created successfully", httpx.SuccessOptions{Data: respdto.<Entity>ROToDTO(&<entity>View)})
}

// GetByID 根据 ID 获取 <Entity>
func (h *<Entity>Handler) GetByID(c fiber.Ctx) error {
    var req reqdto.<Entity>ByIDRequestDTO
    if err := c.Bind().URI(&req); err != nil {
        return errx.ErrInvalidParams.WithCause(err)
    }

    result, err := h.appSvc.Get<Entity>(c.Context(), req.ID)
    if err != nil {
        return err
    }

    return fiberx.OK(c, "<Entity> retrieved successfully", httpx.SuccessOptions{Data: respdto.<Entity>ROToDTO(&result)})
}
```

## HTTP DTO（请求/响应 DTO）

```go
// ✅ GOOD - 请求 DTO 标准结构
package reqdto

import (
    <entity>AppCommands "<project>/modules/<module>/application/<entity>/commands"
    <entity>Domain "<project>/modules/<module>/domain/<entity>"
    "github.com/google/uuid"
)

type <Entity>CreateRequestDTO struct {
    Name   string `json:"name" validate:"required"`
    Status string `json:"status,omitempty"`
}

type <Entity>UpdateStatusRequestDTO struct {
    ID     uuid.UUID `params:"id" validate:"required,uuid"`
    Status string    `json:"status" validate:"required"`
}

type <Entity>ByIDRequestDTO struct {
    ID uuid.UUID `params:"id" validate:"required,uuid"`
}

// ToCreateCmd 转换为创建命令
func (r *<Entity>CreateRequestDTO) ToCreateCmd() <entity>AppCommands.Create<Entity>Cmd {
    cmd := <entity>AppCommands.Create<Entity>Cmd{
        Name: r.Name,
    }

    // 解析状态
    if r.Status != "" {
        cmd.Status = <entity>Domain.<Entity>Status(r.Status)
    } else {
        cmd.Status = <entity>Domain.<Entity>StatusActive // 默认值
    }

    return cmd
}
```

```go
// ✅ GOOD - 响应 DTO 标准结构
package respdto

import (
    <entity>AppResult "<project>/modules/<module>/application/<entity>/results"
)

type <Entity>DTO struct {
    ID        string `json:"id"`
    Name      string `json:"name"`
    Status    string `json:"status"`
    CreatedAt string `json:"created_at"`
    UpdatedAt string `json:"updated_at"`
}

// <Entity>ROToDTO 将应用层结果转换为响应 DTO
func <Entity>ROToDTO(ro *<entity>AppResult.<Entity>RO) *<Entity>DTO {
    if ro == nil {
        return nil
    }

    return &<Entity>DTO{
        ID:        ro.ID.String(),
        Name:      ro.Name,
        Status:    string(ro.Status),
        CreatedAt: ro.CreatedAt.Format("2006-01-02T15:04:05Z07:00"),
        UpdatedAt: ro.UpdatedAt.Format("2006-01-02T15:04:05Z07:00"),
    }
}
```

## HTTP Router（路由）

```go
// ✅ GOOD - 路由标准结构
package http

import (
    "<project>/pkgs/fiberx/middleware"
    "<project>/pkgs/security/token"
    "github.com/gofiber/fiber/v3"
)

type Router struct {
    app           fiber.Router
    tokenVerifier token.Verifier
    handlers      *Registry
}

func NewRouter(app fiber.Router, tokenVerifier token.Verifier, handlers *Registry) *Router {
    return &Router{
        app:           app,
        tokenVerifier: tokenVerifier,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NebulaForgeX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
