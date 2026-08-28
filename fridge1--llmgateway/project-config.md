---
trigger: always_on
description: 本文件为 Codex (Codex.ai/code) 在此代码库中工作时提供指导。
---

# AGENTS.md

本文件为 Codex (Codex.ai/code) 在此代码库中工作时提供指导。

**重要：请始终使用中文与用户交互和回复。**

## 行为准则

通用编码准则：先思考再编码、简洁优先、精准修改、目标驱动执行。

## 项目概述

LLM Gateway 是一个统一的大语言模型 API 网关，支持多种协议格式（OpenAI、Anthropic、Google Gemini），可与 Codex、Cursor、Windsurf、Codex CLI 等 AI 编码工具无缝集成。

**技术栈：**
- 后端：Go 1.26，标准库 HTTP Server，PostgreSQL 16
- 前端：React 19，TypeScript，Vite，Tailwind CSS
- 数据库：PostgreSQL + golang-migrate
- 支付：支付宝
- 短信：火山引擎 SMS

## 开发命令

### 后端

```bash
# 启动后端（默认端口 9090）
go run ./cmd/gateway -config config.yaml

# 运行测试
go test ./...

# 构建二进制文件
go build -o gateway ./cmd/gateway/

# 使用自定义配置和迁移路径运行
go run ./cmd/gateway -config config.local.yaml -migrations ./migrations
```

### 前端

```bash
cd web

# 安装依赖
npm install

# 启动开发服务器（端口 5173，代理到后端 :9090）
npm run dev

# 生产构建
npm run build

# 运行 linter
npm run lint
```

### Docker 生产部署

**当前生产环境使用 Docker Compose 部署，配置文件为 `docker-compose.yml`。**

#### 部署架构

- **backend** 服务（容器名 `llm-gateway`）
  - 使用 `config.docker.yaml` 作为配置文件
  - 敏感信息通过 `.env` 文件注入环境变量
  - 9090 端口不对外暴露，仅内部网络访问
  - 配置健康检查和资源限制（4G 内存 / 4 CPU）
  
- **web** 服务（容器名 `llm-gateway-web`）
  - Nginx 反向代理，监听 80 端口
  - 所有 API 请求（`/api/`, `/v1/`, `/gemini/`, `/v3/`）转发到 backend
  - 前端 SPA 静态文件服务
  - 依赖 backend 健康检查通过后启动

#### 部署命令

```bash
# 完整重新部署（会短暂中断服务）
docker compose down
docker compose up --build -d

# 仅重启服务（不重新构建）
docker compose restart

# 查看服务状态
docker compose ps

# 查看日志
docker compose logs -f backend
docker compose logs -f web
```

#### 配置文件说明

- **`docker-compose.yml`** - Docker Compose 编排配置
- **`config.docker.yaml`** - 后端配置模板（使用环境变量占位符）
- **`.env`** - 环境变量（包含所有敏感信息，不提交到 git）
- **`certs/`** - 支付宝证书目录（挂载到容器）

#### 环境变量清单

`.env` 文件需包含以下变量：
- `JWT_SECRET` - JWT 签名密钥
- `POSTGRES_PASSWORD` - 数据库密码
- `AUTH_TOKEN` - 网关认证 token
- `ADMIN_TOKEN` - 管理员 token
- `SMS_ACCESS_KEY`, `SMS_SECRET_KEY`, `SMS_SIGN_NAME`, `SMS_ACCOUNT` - 短信服务
- `SMS_TPL_LOGIN`, `SMS_TPL_REGISTER`, `SMS_TPL_RESET` - 短信模板 ID
- `ALIPAY_APP_ID`, `ALIPAY_NOTIFY_URL`, `ALIPAY_RETURN_URL`, `ALIPAY_IS_PRODUCTION` - 支付宝配置

#### 数据库配置

生产环境数据库位于 `${DB_HOST}:5432`（内网地址），在 `config.docker.yaml` 中配置：
```yaml
database:
  dsn: "postgres://gateway:${POSTGRES_PASSWORD}@${DB_HOST}:5432/gateway?sslmode=disable"
```

#### 验证部署

```bash
# 检查健康状态
curl http://localhost/health
# 预期输出: {"db":"ok","status":"ok"}

# 检查前端
curl -I http://localhost/
# 预期: HTTP 200

# 检查 API（需要认证）
curl http://localhost/v1/models
# 预期: HTTP 401（未认证）
```

#### 故障排查

如果 web 容器启动失败并报 `host not found in upstream "backend"`：
```bash
# 重启 web 容器（此时 backend 已注册到 DNS）
docker compose restart web
```

如果端口 80 被占用：
```bash
# 查找占用端口的容器
docker ps -a --filter "publish=80"

# 停止并删除旧容器
docker stop <container_id>
docker rm <container_id>
```

### Docker 本地部署

**本地开发/测试使用独立的 Docker Compose，配置文件为 `docker-compose.local.yml`。**

与生产部署完全独立：自带 PostgreSQL 容器，不同的容器名、网络和端口。

#### 部署架构

- **postgres** 服务（容器名 `llm-gateway-local-postgres`）
  - PostgreSQL 16，数据持久化到 Docker volume
  - 仅容器内部访问，不暴露端口到宿主机

- **backend** 服务（容器名 `llm-gateway-local-backend`）
  - 使用 `config.docker.local.yaml` 作为配置文件
  - 环境变量有默认值，零配置可启动

- **web** 服务（容器名 `llm-gateway-local-web`）
  - Nginx 反向代理，监听 3000 端口
  - 使用 `nginx.local.conf` 配置

#### 部署命令

```bash
# 零配置启动（使用内置默认值）
docker compose -f docker-compose.local.yml up --build -d

# 或使用自定义环境变量
cp .env.local.example .env.local
# 编辑 .env.local
docker compose -f docker-compose.local.yml --env-file .env.local up --build -d

# 查看服务状态
docker compose -f docker-compose.local.yml ps

# 查看日志
docker compose -f docker-compose.local.yml logs -f backend

# 停止（保留数据库数据）
docker compose -f docker-compose.local.yml down

# 停止并清除所有数据
docker compose -f docker-compose.local.yml down -v
```

#### 访问方式

- 前端：http://localhost:3000
- 健康检查：http://localhost:3000/health
- 首次注册使用 `ADMIN_INIT_TOKEN`（默认 `your-init-token`）成为管理员

#### 配置文件说明

- **`docker-compose.local.yml`** - 本地 Docker Compose 编排配置
- **`config.docker.local.yaml`** - 本地后端配置模板（使用环境变量占位符）
- **`.env.local.example`** - 环境变量示例文件
- **`.env.local`** - 本地环境变量（不提交到 git）

## 架构设计

### 核心组件

**代理层** (`internal/proxy/`, `internal/anthropic/`, `internal/responses/`, `internal/gemini/`)
- 处理多种格式的传入 API 请求
- 路由到合适的上游提供商
- 管理认证、计费和响应流式传输

**路由器** (`internal/router/`)
- 将模型名称映射到上游提供商池
- 支持模型别名（规范名称 vs 显示名称）
- 集成熔断器实现故障转移

**适配器** (`internal/adapter/`)
- 在 API 格式之间转换（OpenAI ↔ Anthropic）
- 实现协议无关的上游路由

**计费** (`internal/billing/`)
- 预扣费计费模型
- 通过工作池异步结算
- 支持余额、赠送金和佣金账户
- 基于 token 的定价，支持缓存定价

**存储** (`internal/store/`)
- PostgreSQL 数据访问层
- 管理用户、API 密钥、交易、模型、定价、订阅、租户

**配置** (`internal/config/`)
- 数据库驱动的模型和上游配置
- YAML 配置仅用于服务器设置
- 通过 `rebuildRouter()` 回调支持热重载

### API 端点

**公开代理端点：**
- `POST /v1/chat/completions` - OpenAI Chat Completions（Cursor、Windsurf、Aider）
- `POST /v1/messages` - Anthropic Messages（Codex）
- `POST /v1/responses` - OpenAI Responses（Codex CLI）
- `POST /gemini/v1/*` - Google Gemini 原生格式
- `GET /v1/models` - 列出可用模型（OpenAI 兼容）

**管理员端点**（需要 JWT + admin 角色）：
- `/api/admin/users` - 用户管理
- `/api/admin/pricing` - 定价配置
- `/api/admin/managed-keys` - 托管 API 密钥
- `/api/admin/dashboard` - 分析仪表板

**用户端点**（需要 JWT）：
- `/api/keys` - API 密钥管理
- `/api/billing/*` - 余额、交易、统计
- `/api/subscription/*` - 订阅计划和使用情况
- `/api/payment/*` - 支付宝支付操作

### 认证流程

1. 从 `Authorization` 头或 `x-api-key` 头提取 Bearer token
2. 使用 SHA256 哈希 token
3. 检查缓存 → 用户密钥 → 租户密钥 → 托管密钥
4. 验证密钥状态和余额
5. 将认证上下文附加到请求

### 计费流程


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fridge1/llmgateway](https://github.com/fridge1/llmgateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
