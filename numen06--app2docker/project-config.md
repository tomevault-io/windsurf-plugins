---
trigger: always_on
description: app2docker 项目核心架构与构建流程上下文
---


⚠️ 本文件由 AI 自动生成，项目结构变更后请重新执行生成命令

## 项目概述

App2Docker：可视化 Docker 镜像构建平台。FastAPI 后端 + Vue3 前端，单容器部署（挂载 `data/` + `docker.sock`）。支持文件上传/Git 源码构建、流水线、镜像导出、SSH/Agent/Portainer 部署。

当前版本：`backend/VERSION` → **1.0.2**；前端 `frontend/package.json` → **1.0.2**

---

## 【技术栈】

### 后端（Python 3）
| 依赖 | 版本 | 用途 |
|------|------|------|
| fastapi | 0.115.6 | HTTP API |
| uvicorn[standard] | 0.34.0 | ASGI 服务 |
| sqlalchemy | 2.0.23 | ORM（SQLite） |
| docker | 7.1.0 | Docker API / buildx |
| PyJWT | 2.10.1 | JWT 认证 |
| PyYAML | 6.0.3 | 配置/部署 YAML |
| paramiko | 3.4.0 | SSH 部署 |
| httpx | 0.27.0 | Webhook 触发 |
| websockets | 14.1 | Agent WebSocket 客户端 |
| psutil | 6.1.0 | Agent 主机信息采集 |
| croniter | 5.0.1 | 流水线定时调度 |
| watchdog | 5.0.3 | 文件监听 |

入口：`backend/app.py`（`python backend/app.py` 或 Dockerfile `CMD`）

### 前端（Node 20 + Vue 3）
| 依赖 | 版本 | 用途 |
|------|------|------|
| vue | ^3.5.24 | UI 框架 |
| vite | ^5.4.0 | 构建/开发服务器（port 3000，proxy `/api` → 8000） |
| axios | ^1.13.2 | HTTP 客户端 |
| bootstrap | ^5.3.8 | UI 组件 |
| tailwindcss | ^4.2.2 | 样式 |
| jwt-decode | ^4.0.0 | 前端 JWT 解析 |
| codemirror / vue-codemirror | ^6.x | YAML/Dockerfile 编辑器 |

构建产物：`frontend/vite.config.js:37-40` → `outDir: '../dist'`

### 基础设施
- 数据库：SQLite `data/app2docker.db`（WAL 模式，`backend/database.py:12-14`）
- 运行时数据：`data/`（config.yml、uploads、docker_build、exports、templates、logs、secret_key.txt）
- 内置模板：`templates/{jar,nodejs,python,go,web}/`
- Docker 镜像：`Dockerfile` 多阶段（frontend-builder → backend-base → app2docker / app2docker-agent）

---

## 【目录结构】

> 本项目无根级 `src/`；前端源码在 `frontend/src/`，后端在 `backend/`。

```
frontend/src/                    # Vue 前端（3层）
├── App.vue                      # 根组件、菜单路由
├── main.js                      # 入口，挂载 axios 拦截器
├── components/                  # 功能面板（按菜单划分）
│   ├── StepBuildPanel.vue       # 镜像构建
│   ├── PipelinePanel.vue        # 流水线
│   ├── DataSourcePanel.vue      # Git 数据源
│   ├── TaskManager.vue          # 任务管理
│   ├── TemplatePanel.vue        # 模板管理
│   ├── DeployTaskManager.vue    # 部署管理
│   ├── AgentHostManager.vue     # Agent 主机
│   ├── HostManager.vue          # SSH 主机
│   ├── PortainerDeployManager.vue
│   ├── LoginPage.vue / UserManagement.vue / RoleManagement.vue
│   └── ...                      # Dashboard、Export、Registry、Docker 等
├── composables/
│   └── useModalEscape.js
└── utils/
    ├── auth.js                  # Token 存取
    ├── axios-interceptor.js     # 401 跳转登录
    ├── permissions.js           # 前端权限过滤菜单
    └── projectTypes.js          # 项目类型（与 backend/project_types.py 同步）

backend/                         # Python 后端（非 src/）
├── app.py                       # FastAPI 入口、startup 初始化
├── routes.py                    # 全部 REST + WebSocket 路由（/api 前缀）
├── handlers.py                  # 构建/导出/模板核心业务
├── docker_builder.py            # Local/Remote/Mock Docker 构建器
├── config.py                    # data/config.yml 读写
├── auth.py                      # JWT + 用户/权限
├── database.py / models.py      # SQLite ORM
├── git_source_manager.py        # Git 数据源 CRUD
├── pipeline_manager.py          # 流水线
├── scheduler.py                 # cron 调度
├── template_parser.py           # {{VAR}} 模板变量替换
├── project_types.py             # jar/nodejs/python/go/web
├── websocket_handler.py         # Agent WS 连接管理
├── deploy_executors/            # agent / ssh / portainer 部署执行器
│   └── factory.py
└── agent/                       # 远程 Agent 进程
    ├── main.py
    └── websocket_client.py

# 省略：node_modules/、dist/、.git/、data/（运行时）、docs/、test/、scripts/
templates/                       # 内置 Dockerfile 模板（只读）
├── jar/                         # dragonwell8/17/21 build+upload
├── nodejs/                      # nodejs18/20
├── python/                      # python39/310/311/312
├── go/                          # go1.21/22/23
└── web/                         # nginx-simple
```

---

## 【Docker 镜像构建流程】

### A. App2Docker 自身镜像（平台容器）

```
Dockerfile
├── 阶段1 frontend-builder (L5-35)
│   COPY frontend/ → npm install → npm run build → /app/dist
├── 阶段2 backend-base (L39-113)
│   Alpine + Python venv + requirements.txt + COPY backend/
├── 阶段3 app2docker-agent (L116-148, --target app2docker-agent)
│   CMD backend/agent/start.sh
└── 阶段4 app2docker 默认 (L151-194)
    COPY --from=frontend-builder /app/dist → ./dist
    COPY templates/ → ./templates/
    ENV APP_PORT=8000
    CMD python backend/app.py
```

独立 Agent 镜像：`agent.Dockerfile`（等价于主 Dockerfile 的 agent 阶段）

### B. 用户应用镜像构建（核心业务链）

#### 路径1：文件上传构建
```
POST /api/upload                          routes.py:2019
  → BuildManager.start_build()            handlers.py:1410
  → 后台线程 _build_task()                handlers.py:1464
      ├── 写入 data/uploads + 解压到 data/docker_build/{image}_{taskId}
      ├── get_template_path()             handlers.py:177 → templates/ 或 data/templates/
      ├── parse_template() 替换 {{VAR}}   template_parser.py:112 + handlers.py:2840
      ├── create_docker_builder()         docker_builder.py:1102
      │     ├── use_remote=true → RemoteDockerBuilder (L622)
      │     ├── 否则 LocalDockerBuilder (L463) → docker buildx build
      │     └── 均不可用 → MockDockerBuilder (L1050)
      └── docker_builder.build_image()    handlers.py:1937
            → _build_with_buildx()        docker_builder.py:178-460
            → 可选 push_image()           handlers.py:2166
```

#### 路径2：Git 源码构建
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numen06/app2docker](https://github.com/numen06/app2docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
