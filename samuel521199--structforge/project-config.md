---
trigger: always_on
description: StructForge 是一个全功能的AI工作流编排平台，支持多种AI大模型（开源、自建、公共API）和各种工作节点，提供可视化的流程设计界面。类似n8n，但专注于AI工作流场景。
---


# StructForge 项目开发规则

## 1. 项目目标与定位

### 1.1 项目目标

StructForge 是一个全功能的AI工作流编排平台，支持多种AI大模型（开源、自建、公共API）和各种工作节点，提供可视化的流程设计界面。类似n8n，但专注于AI工作流场景。

### 1.2 核心特性

- 多AI模型支持：支持OpenAI、Gemini、Ollama等各类AI模型
- 丰富的工作节点：触发、AI、数据处理、集成、控制、工具等节点
- 可视化编辑器：拖拽式工作流设计，直观易用
- 高性能执行：支持同步/异步执行，适合各种场景
- 安全可靠：完善的认证授权、数据加密、多租户隔离
- 易于扩展：插件化架构，支持自定义节点和模型

## 2. 技术架构

### 2.1 前端技术栈

- **框架**: Vue 3.x (Composition API)
- **语言**: TypeScript
- **构建工具**: Vite
- **状态管理**: Pinia
- **UI组件库**: Element Plus
- **工作流编辑器**: Vue Flow
- **HTTP客户端**: Axios
- **WebSocket**: Socket.io-client 或原生 WebSocket

### 2.2 后端技术栈

- **框架**: Go + Kratos微服务框架
- **协议**: gRPC + HTTP Gateway
- **数据库**: PostgreSQL (主数据库) + Redis (缓存/队列)
- **消息队列**: RabbitMQ (可选，用于异步任务)
- **服务发现**: Consul / etcd (Kratos内置支持)
- **配置中心**: Kratos配置系统
- **日志**: 统一使用 `backend/common/log` 日志系统
- **链路追踪**: OpenTelemetry (可选)

### 2.3 基础设施

- **容器化**: Docker + Docker Compose
- **CI/CD**: GitHub Actions / GitLab CI
- **监控**: Prometheus + Grafana (可选)
- **平台支持**: Windows、Linux

### 2.4 文档系统

- **API文档**: OpenAPI 3.0 + Swagger UI（集成到API Gateway）
- **节点说明**: 数据库存储 + API提供（集成到节点服务）
- **工作流说明**: Markdown文件 + API提供（集成到工作流服务）
- **独立文档站**: VitePress（可选，用于对外文档）

## 3. 项目结构

### 3.1 根目录结构

```text
StructForge/
├── frontend/                 # 前端项目
├── backend/                  # 后端项目（Go + Kratos）
│   ├── api/                  # API定义（Protobuf）
│   ├── apps/                 # 微服务应用
│   │   ├── gateway/          # API Gateway服务
│   │   ├── user/             # 用户服务
│   │   ├── workflow/         # 工作流服务
│   │   ├── ai/               # AI模型服务
│   │   ├── node/             # 节点服务
│   │   ├── tool/             # 工具服务
│   │   ├── scheduler/        # 调度服务
│   │   └── log/              # 日志服务
│   ├── common/               # 公共代码
│   ├── configs/              # 配置文件
│   ├── deploy/               # 部署相关
│   ├── script/               # 开发脚本
│   ├── third_party/          # 第三方依赖
│   ├── go.mod                # Go模块定义
│   └── Makefile              # Make命令
├── docs/                     # 项目文档
├── docker/                   # Docker相关配置
└── docker-compose.yml        # 本地开发环境配置
```

### 3.2 前端项目结构

```text
frontend/
├── src/
│   ├── api/                  # API接口定义
│   ├── assets/               # 资源文件
│   ├── components/           # 公共组件
│   │   ├── common/           # 通用组件
│   │   ├── workflow/         # 工作流相关组件
│   │   └── layout/           # 布局组件
│   ├── composables/          # Composition API组合函数
│   ├── router/               # 路由配置
│   ├── stores/               # Pinia状态管理
│   ├── utils/                # 工具函数
│   └── views/                # 页面视图
```

### 3.3 后端服务结构（Kratos规范）

```text
apps/service-name/
├── cmd/
│   └── service-name/          # 启动入口
│       ├── main.go
│       └── wire.go
└── internal/                 # 内部实现
    ├── biz/                  # 业务逻辑层（核心）
    │   ├── service-name.go   # 核心业务逻辑
    │   └── ...               # 其他业务逻辑文件
    ├── data/                 # 数据访问层
    │   ├── service-name.go   # 数据访问实现
    │   └── db.go             # 数据库连接
    ├── service/              # 服务层（gRPC实现）
    │   └── service-name.go   # gRPC服务实现
    ├── server/               # 服务器配置
    │   ├── grpc.go
    │   └── http.go
    └── conf/                 # 配置
        └── config.yaml       # 服务配置

# API定义在 backend/api/service-name/v1/ 目录
backend/api/
└── service-name/
    └── v1/
        └── service-name.proto
```

### 3.5 后端特殊目录说明

- **backend/apps/ai/internal/adapters/**: AI模型适配器实现目录
  - `openai/`: OpenAI适配器
  - `gemini/`: Gemini适配器
  - `ollama/`: Ollama适配器
  - `custom/`: 自定义模型适配器

- **backend/apps/node/internal/nodes/**: 节点实现目录
  - `trigger/`: 触发节点（webhook.go, timer.go, manual.go）
  - `ai/`: AI节点（text_generation.go, image_generation.go）
  - `data/`: 数据处理节点（transform.go, filter.go）
  - `integration/`: 集成节点（http.go, database.go）
  - `control/`: 控制节点（condition.go, loop.go）
  - `tool/`: 工具节点（script.go）

- **backend/common/**: 跨服务公共代码
  - `middleware/`: 中间件（auth, cors, logger, ratelimit）
  - `data/`: 数据访问公共代码（database, redis）
  - `utils/`: 工具函数（crypto, response, validator）
  - `log/`: **统一日志系统**（所有后端服务必须使用此日志系统，禁止使用其他日志库）

- **backend/apps/gateway/internal/handler/**: API Gateway HTTP处理器
  - HTTP请求处理
  - 路由转发
  - 认证授权

- **backend/api/**: 所有服务的Protobuf定义
  - `user/v1/user.proto`: 用户服务API定义
  - `workflow/v1/workflow.proto`: 工作流服务API定义
  - `ai/v1/ai.proto`: AI模型服务API定义
  - `node/v1/node.proto`: 节点服务API定义
  - `gateway/v1/gateway.proto`: Gateway服务API定义
  - `common/v1/common.proto`: 公共定义

### 3.4 微服务划分

- **apps/gateway**: API Gateway，统一HTTP入口，路由、认证、限流
- **apps/user**: 用户注册、登录、授权、权限管理、组织管理
- **apps/workflow**: 工作流定义、版本控制、执行引擎、状态管理
- **apps/ai**: AI模型统一接入、配置管理、调用封装、限流监控
- **apps/node**: 各种工作流节点实现、节点类型管理、参数验证
- **apps/tool**: 外部工具集成、工具配置管理、工具调用封装
- **apps/scheduler**: 定时任务调度、工作流定时执行、任务队列管理
- **apps/log**: 工作流执行日志收集、系统日志聚合、日志查询分析

## 4. 编程命名规范

### 4.1 服务命名

- **服务目录名**: 小写字母，无连字符
  - 示例: `user`, `workflow`, `ai`, `gateway`
- **服务标识**: 在代码中使用时可以用kebab-case或snake_case
  - 示例: `user-service`, `workflow-service`, `ai-service`

### 4.2 Go语言命名规范

- **包名**: 小写字母，无下划线，简短有意义
  - 示例: `user`, `workflow`, `ai`
- **文件名**: snake_case
  - 示例: `user.go`, `workflow_execution.go`, `ai_adapter.go`
- **结构体名**: PascalCase
  - 示例: `User`, `WorkflowExecution`, `AIModelAdapter`
- **函数/方法名**:
  - 公开函数/方法: PascalCase（首字母大写）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samuel521199) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
