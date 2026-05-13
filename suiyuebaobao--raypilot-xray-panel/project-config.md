---
trigger: always_on
description: 本文件是 RayPilot 仓库唯一规则文件。原其他规则文件的有效项已整合到本文，后续不再维护多份规则，也不再要求跨规则文件同步。
---

# AGENTS.md

本文件是 RayPilot 仓库唯一规则文件。原其他规则文件的有效项已整合到本文，后续不再维护多份规则，也不再要求跨规则文件同步。

## 项目概述

RayPilot 是一套面向 `xray-core` 节点的订阅分发、用户管理和中转管理系统。用户购买套餐后获得订阅链接，通过 Clash/mihomo、Shadowrocket、Surge 等客户端连接代理节点。节点控制面通过独立部署的 `node-agent` 管理本机 `xray-core` 的用户权限、流量上报、健康状态和配置任务。

完整开发方案详见 `开发方案.md`。当前状态为 v1 功能开发完成，已进入测试与运维阶段；第一版直连与中转并存能力已落地，包含 `/admin/relays`、node-agent relay 模式和 HAProxy 配置下发。

## 核心架构

- 数据面：`xray-core` 负责实际代理流量转发，主要协议为 VLESS + Reality，传输层支持 TCP 与 XHTTP。
- 控制面：`node-agent` 负责管理本机 `xray-core` 用户、Xray/HAProxy 配置、流量上报、状态上报和任务执行。
- 中心服务：对出口节点只和 node-agent exit/multi_exit 模式通信，不直接跨公网调用 xray-core gRPC；中转阶段对接 node-agent relay 模式。
- 订阅分发：HTTPS 下载订阅，唯一入口为 `/sub/{token}`，只下发 Clash/mihomo YAML；`/sub/{token}/clash`、`/sub/{token}/base64` 与 `/sub/{token}/plain` 等带后缀入口全部下线。
- 中转能力：node-agent relay 模式和 TCP 透传中转层已落地，直连出口节点继续保留。
- 流量计费：快照差值法，每次上报存储累计值，与上次快照求差得到增量，避免重复计费。
- 任务管理：MySQL 任务表 + 乐观锁 `lock_token`，v1 不引入 Redis 或 BullMQ。

## 技术栈

| 层 | 技术 |
| --- | --- |
| 后端 | Go + Gin + GORM + MySQL 8.0+ |
| 前端 | Vue 3 + Vite + Element Plus + Alova + Pinia + Vue Router 4 |
| 部署 | Docker Compose + Nginx 反代 |
| 鉴权 | JWT Access Token + Refresh Token |
| 任务调度 | robfig/cron/v3 + MySQL 任务表 |
| 节点代理 | xray-core + node-agent，relay 模式默认 HAProxy |

## 项目结构与模块组织

本仓库包含 Go 后端、Vue 前端、部署配置和中文项目文档。

- `cmd/api`：Gin HTTP API 服务入口，默认端口 3000。
- `cmd/worker`：后台定时任务 Worker 入口。
- `cmd/seed`：数据库种子工具，创建初始管理员。
- `cmd/node-agent`：真实节点代理，管理本机 xray-core，支持 exit、multi_exit、relay 模式。
- `cmd/node-agent-mock`：开发用节点代理模拟程序。
- `internal/`：后端核心包，包括 handler、service、repository、model、middleware、scheduler、subscription、config、auth、database、response、agent、platform。
- `migrations/`：SQL 数据库迁移，表结构以 migration 为准，不依赖 GORM 自动迁移。
- `frontend/`：Vue 3 + Vite + Element Plus 前端项目，源码在 `frontend/src`。
- `web/static/`：前端生产构建产物。
- `deploy/nginx/`：Nginx 反向代理配置。
- `文档/`：架构、接口、部署、运维、页面清单、测试报告和每日记录。
- `开发方案.md`：完整开发方案。

Go 测试文件与实现文件同目录，命名为 `*_test.go`。前端页面主要位于 `frontend/src/pages/user` 和 `frontend/src/pages/admin`。

## 后端分层

后端遵循 `HTTP Request -> Handler -> Service -> Repository -> MySQL` 的分层。

- Handler 层位于 `internal/handler/`，只负责参数解析、调用 Service 和写响应，不写业务逻辑。
- Service 层位于 `internal/service/`，承载业务规则、事务、权限和跨 Repository 编排，错误返回 AppError。
- Repository 层位于 `internal/repository/`，封装 GORM 数据访问。
- Model 层位于 `internal/model/`，维护 GORM 模型、请求结构和响应结构。
- Middleware 层位于 `internal/middleware/`，处理鉴权、日志、跨域等横切逻辑。
- Subscription 层位于 `internal/subscription/`，负责 Clash/mihomo YAML 订阅输出。

## 前端与路由

前端使用 Vue 3 + Vite + Element Plus + Alova + Pinia + Vue Router 4。

- 用户路由包括 `/login`、`/register`、`/`、`/subscription`、`/orders`、`/plans`、`/redeem`、`/profile`。
- 管理后台路由包括 `/admin/login`、`/admin`、`/admin/plans`、`/admin/node-groups`、`/admin/nodes`、`/admin/relays`、`/admin/users`、`/admin/orders`、`/admin/redeem-codes`、`/admin/subscription-tokens`、`/admin/logs`、`/admin/node-operations`、`/admin/subscription-settings`。
- 路由守卫通过 `requiresAuth`、`requiresAdmin`、`guest` meta 标记和 `beforeEach` 执行。
- 接口调用应通过 `frontend/src/api/request.js` 或专门 API 适配层收口，避免页面内散落原生 `fetch`。

## 鉴权机制

系统使用 JWT 双 Token。

- Access Token 放在 `Authorization: Bearer` 请求头，默认 24 小时过期。
- Refresh Token 放在 HttpOnly Cookie `refresh_token`，默认 7 天过期。
- 刷新和退出必须使旧 Refresh Token 失效，避免长期可复用。
- API 统一返回 `{success, message, code, data}` 格式。

## 核心数据表

当前核心表包括 `users`、`plans`、`node_groups`、`plan_node_groups`、`nodes`、`node_hosts`、`user_subscriptions`、`subscription_tokens`、`refresh_tokens`、`orders`、`payment_records`、`traffic_snapshots`、`node_access_tasks`、`usage_ledgers`、`redeem_codes`、`relays`、`relay_backends`、`relay_config_tasks`、`relay_traffic_snapshots`、`site_settings`、`operation_logs`、`deployment_logs`、`node_runtime_metrics`、`node_health_checks`。

数据库迁移文件位于 `migrations/*.up.sql`，使用 golang-migrate 执行。

## 构建、测试与开发命令

- `cp .env.example .env`：首次配置环境变量。
- `docker-compose up -d mysql`：启动 MySQL。
- `make migrate`：基于 `MIGRATE_DATABASE_URL` 执行 SQL 迁移。
- `make api`：启动 API 服务，即 `go run ./cmd/api`。
- `make worker`：启动后台定时任务。
- `make seed`：运行数据库种子工具。
- `make frontend`：启动 Vite 前端开发服务器。
- `make frontend-build`：构建前端产物到 `web/static`。
- `go test ./...`：运行全部后端测试。
- `go test ./internal/service/... -v`：运行指定 service 包测试。
- `cd frontend && npm run build`：验证前端生产构建。
- `make build`：构建 Go 二进制到 `bin/`。
- `make docker`：执行 `docker-compose build`。
- `make up` / `make down`：启动或停止 Docker Compose 服务。
- `docker-compose config --services`：校验 Compose 服务配置。

当前运行环境使用 `docker-compose` 命令；Makefile 默认 `COMPOSE ?= docker-compose`，如目标环境只支持 Compose v2 子命令，可显式执行 `COMPOSE="docker compose" make up`。

## 编码风格与命名规范

- Go 代码必须使用 `gofmt`。
- 包名保持短小、全小写。
- 每个代码文件前 10 行内写功能简介。
- 复杂业务逻辑、事务处理、并发控制等代码应添加必要注释，但不要添加无意义注释。
- 统一错误处理使用 `platform/response.AppError`。
- Handler 不写业务逻辑，只做参数解析、Service 调用、响应写入。
- Service 层事务操作返回 AppError。
- v1 不引入 Redis。
- 前端使用 Vue 单文件组件、Composition API 和 Element Plus，缩进为 2 个空格；现有 JavaScript 风格不使用分号。
- 测试命名优先使用 `Test{功能}_{场景}_{预期}`，例如 `TestAuthService_Register_Success`。

## 测试要求

- 后端测试使用 Go testing 与 `stretchr/testify`。
- 修改 service、repository、handler、middleware、subscription、scheduler 等核心包时，应补充对应测试。
- Service 层优先写单元测试，Repository 层写集成测试，Handler 层写 HTTP 端到端测试。
- 涉及事务、鉴权、token、流量计费、节点任务处理的改动，需要增加聚焦测试。
- 每完成一项功能开发或 bug 修复，应立即编写对应测试并运行通过，然后再进行下一项工作。
- 提交前至少运行 `go test ./...`，核心业务路径目标覆盖率不低于 80%。
- 涉及前端改动时运行 `cd frontend && npm run build`。
- 涉及关键用户路径、节点部署、订阅输出、管理后台页面时运行 Playwright smoke。

## 文档要求


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suiyuebaobao/raypilot-xray-panel](https://github.com/suiyuebaobao/raypilot-xray-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
