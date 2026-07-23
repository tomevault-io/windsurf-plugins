---
trigger: always_on
description: A dedicated guide for AI coding agents working on the Verdure MCP Platform project.
---

﻿# AGENTS.md

A dedicated guide for AI coding agents working on the Verdure MCP Platform project.

**This file follows the [agents.md](https://agents.md/) specification - a standard format for providing AI coding agents with project-specific context, conventions, and instructions.**

---

## 🎯 Quick Start for Agents

### Essential Commands
```powershell
# Restore dependencies
dotnet restore

# Build entire solution
dotnet build

# Run with Aspire orchestration (recommended)
dotnet run --project src/Verdure.McpPlatform.AppHost

# Run API only
dotnet run --project src/Verdure.McpPlatform.Api

# Run Web frontend only
dotnet run --project src/Verdure.McpPlatform.Web

# Run all tests
dotnet test

# Database migrations (EF Core)
dotnet ef migrations add <MigrationName> --project src/Verdure.McpPlatform.Infrastructure --startup-project src/Verdure.McpPlatform.Api
dotnet ef database update --project src/Verdure.McpPlatform.Infrastructure --startup-project src/Verdure.McpPlatform.Api
```

### Testing Commands
```powershell
# Unit tests only
dotnet test tests/Verdure.McpPlatform.UnitTests

# Functional tests only
dotnet test tests/Verdure.McpPlatform.FunctionalTests

# Run with coverage
dotnet test /p:CollectCoverage=true

# Watch mode during development
dotnet watch test --project tests/Verdure.McpPlatform.UnitTests
```

### Verification Script
```powershell
# Verify complete setup
.\scripts\verify-setup.ps1

# Start development environment
.\scripts\start-dev.ps1
```

---

## 📖 项目概述 (Project Overview)

**Verdure MCP Platform** 是一个基于 .NET 10 的企业级多租户 SaaS 平台，为小智 AI 助手提供完整的 Model Context Protocol (MCP) 服务管理解决方案。

### 核心功能 (Core Features)

1. **多租户身份认证** - 基于 Keycloak OpenID Connect
2. **小智连接管理** - 配置小智 AI 服务器的 WebSocket 端点地址
3. **MCP 服务配置** - 管理各类 MCP 服务节点及其工具
4. **服务绑定** - 将 MCP 服务动态绑定到小智连接节点
5. **分布式 WebSocket 管理** - 支持多实例部署的 WebSocket 连接协调
6. **自动重连机制** - 后台监控和自动恢复断开的连接

### 架构模式 (Architecture Patterns)

- **前后端分离** - Blazor WebAssembly SPA + ASP.NET Core Web API
- **领域驱动设计 (DDD)** - 清晰的领域模型和聚合根
- **仓储模式 (Repository Pattern)** - 数据访问抽象层
- **依赖注入 (Dependency Injection)** - 完全基于 ASP.NET Core DI
- **分布式协调** - 使用 Redis 实现跨实例状态管理和分布式锁

---

## 🛠️ 技术栈 (Tech Stack)

### 后端 (Backend)
- **.NET 10** - 最新的 .NET 平台
- **ASP.NET Core Web API** - RESTful API with Minimal APIs pattern
- **ASP.NET Core Identity** - 用户认证和授权
- **OpenID Connect** - Keycloak 集成
- **Entity Framework Core 9.0** - ORM 框架，支持自动迁移
- **Redis** - 分布式缓存、状态管理和分布式锁
- **PostgreSQL / SQLite** - 支持多数据库切换

### 前端 (Frontend)
- **Blazor WebAssembly** - 客户端 SPA，离线支持
- **MudBlazor** - Material Design 3 UI 组件库
- **国际化 (i18n)** - 完整的多语言支持（中文/英文）

### 服务编排 (Orchestration)
- **.NET Aspire** - 云原生应用编排和开发仪表板

### 分布式系统 (Distributed System)
- **StackExchange.Redis 2.9.32** - Redis 客户端
- **RedLock.net 2.3.2** - 分布式锁（RedLock 算法）
- **WebSocket** - 实时双向通信
- **Background Services** - 连接监控和自动重连

### 核心 NuGet 包
```xml
<!-- MCP Protocol -->
<PackageReference Include="ModelContextProtocol" Version="0.3.0-preview.3" />
<PackageReference Include="ModelContextProtocol.AspNetCore" Version="0.3.0-preview.3" />

<!-- Identity & Authentication -->
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="10.0.*" />
<PackageReference Include="Microsoft.AspNetCore.Authentication.OpenIdConnect" Version="10.0.*" />

<!-- Database -->
<PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="10.0.*" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="10.0.*" />

<!-- Redis & Distributed Coordination -->
<PackageReference Include="StackExchange.Redis" Version="2.9.32" />
<PackageReference Include="RedLock.net" Version="2.3.2" />

<!-- Blazor & UI -->
<PackageReference Include="MudBlazor" Version="8.*" />
```

---

## 📚 参考项目 (Reference Projects)

生成代码时，必须严格参考以下项目的架构和代码风格：

### 1. 架构和代码风格参考（最高优先级）
**仓库**: https://github.com/dotnet/eShop
- **必须严格遵循 eShop 的架构模式和代码风格**
- 仓储模式 (Repository Pattern) 实现
- 服务层 (Service Layer) 设计
- 依赖注入最佳实践
- 身份认证和授权实现（ASP.NET Core Identity + OpenID Connect）
- 领域驱动设计 (DDD) 模式
- Entity Framework Core 最佳实践

**关键参考点**：
- `src/Ordering.Infrastructure/Repositories/` - 仓储实现
- `src/Ordering.Domain/SeedWork/IRepository.cs` - 仓储接口
- `src/Identity.API/` - Identity 服务器实现
- `src/WebApp/Extensions/Extensions.cs` - OpenID Connect 配置
- `src/eShop.ServiceDefaults/AuthenticationExtensions.cs` - 认证扩展
- `src/Basket.API/Extensions/Extensions.cs` - 依赖注入模式
- `src/Ordering.API/Extensions/Extensions.cs` - 应用服务注册

### 2. MCP 协议实现参考
**仓库**: https://github.com/maker-community/mcp-calculator/tree/dev_csharp/csharp
- MCP 协议的 C# 实现方式
- WebSocket 连接和服务绑定逻辑

### 3. 前后端集成参考
**仓库**: https://github.com/GreenShadeZhang/agent-framework-tutorial-code/tree/main/agent-groupchat
- Blazor 与 ASP.NET Core API 的集成方式
- .NET Aspire 服务编排配置

---

## 📂 项目文档结构 (Documentation Structure)

项目采用结构化文档组织，所有文档位于 `docs/` 目录：

### 架构文档 (docs/architecture/)
- `DISTRIBUTED_WEBSOCKET_GUIDE.md` - 分布式 WebSocket 连接管理详细指南
- `FAILURE_RECOVERY_EXPLAINED.md` - 故障恢复机制说明
- `IMPLEMENTATION_SUMMARY.md` - 分布式实现总结
- `WEBSOCKET_FEATURES.md` - WebSocket 功能特性
- `MCP_AUTH_ENHANCEMENT.md` - MCP 认证增强方案
- `AGENTS.md` / `AGENTS_NEW.md` - AI 编程助手指南

### 开发指南 (docs/guides/)
- `QUICK_START_DISTRIBUTED.md` - 分布式部署快速开始
- `API_EXAMPLES.md` - API 使用示例

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maker-community/verdure-mcp-for-xiaozhi](https://github.com/maker-community/verdure-mcp-for-xiaozhi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
