---
trigger: always_on
description: 本文档面向在本仓库中工作的编码 Agent / 开发者，说明项目目标、目录约定、技术选型与实现边界。修改代码前请先阅读。
---

# AGENTS.md — zacp 项目协作指南

本文档面向在本仓库中工作的编码 Agent / 开发者，说明项目目标、目录约定、技术选型与实现边界。修改代码前请先阅读。

---

## 1. 项目是什么

**zacp** 是一个 **ACP（Agent Client Protocol）多 Agent Web 网关**：

- 通过 Web UI 接入多种支持 ACP 协议的 Agent 工具（如 Pi Agent、Reasonix、Grok 等）
- 后端以 **ACP Client** 身份连接各 Agent（本地 stdio 子进程或远程通道）
- 向前端提供 HTTP API + WebSocket，用于会话管理、消息流式输出、权限确认等

协议文档：https://agentclientprotocol.com  
Go SDK：https://github.com/coder/acp-go-sdk（模块路径 `github.com/coder/acp-go-sdk`）

---

## 2. 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| 后端 | Go（当前 go.mod 为 1.25.x） | 模块名 `github.com/helloxz/zacp` |
| HTTP | Gin `v1.12.x` | REST API、路由、中间件 |
| WebSocket | **`github.com/coder/websocket`** | 浏览器会话主通道（流式输出、权限、取消）；实现放在 `internal/ws` |
| 配置 | **TOML + Viper** | 运行时配置 **`~/.zacp/config.toml`**；库 **`github.com/spf13/viper`**；加载在 `internal/config` |
| 数据库 | **SQLite3 + GORM + 纯 Go 驱动** | ORM：`gorm.io/gorm`；驱动：`github.com/glebarez/sqlite`（底层 `modernc.org/sqlite`，**无 CGO**）；持久化在 `internal/store` |
| ACP | `github.com/coder/acp-go-sdk` | Client 侧连接、会话、Prompt、SessionUpdate |
| 前端 | **Vue 3 + Naive UI + Tailwind CSS** | 代码在 `frontend/`；构建建议 Vite；**包管理与脚本一律用 Bun**；实时通道用浏览器原生 `WebSocket` |
| 部署 | 根目录`scripts/` | 运维脚本 |

**实时通信选型（已定）：**

- **选用 WebSocket**，不用 SSE 作为会话主通道（连续对话 + 权限回传 + 取消需要双向）。
- **Go 库固定为 `github.com/coder/websocket`**（原 `nhooyr/websocket`）。不要使用Socket.IO 等替代实现，除非有充分理由并先更新本文档。
- Gin 侧在 handler 内 `websocket.Accept` 升级连接；升级后勿再写 `c.JSON`。
- REST 仍用于健康检查、agent 列表、配置等非实时接口；会话实时交互走 WS。

**配置选型（已定）：**

- **主配置格式：TOML**。不要改用 YAML/JSON/INI 作为主配置，除非先更新本文档。
- **配置库：`github.com/spf13/viper`**（读 TOML、默认值、环境变量覆盖、可选写回）。
- 实现集中在 `backend/internal/config`：对外强类型 `Config`；业务层 **不要** 散落 `viper.Get*`。
- 密钥、Token **不进** TOML；用环境变量（或本机 `.env`，gitignore）注入；Agent 子进程继承环境。

**数据存储选型（已定）：**

- **SQLite3** 作为默认（也是当前唯一规划的）嵌入式数据库；
- **ORM：GORM**（`gorm.io/gorm`）。
- **驱动：纯 Go**——通过 **`github.com/glebarez/sqlite`** 使用 **`modernc.org/sqlite`**，**禁止**默认依赖需要 CGO 的 `mattn/go-sqlite3`（除非有充分理由并更新本文档）。
- **数据文件目录：`$ZACP_DATA/data/`**（默认 `~/.zacp/data/`）。
- 实现集中在 `backend/internal/store`（打开 DB、PRAGMA、迁移、仓储）；业务通过 store/repository 访问，不在 handler 里直接拼 SQL。


---

## 3. 目录结构（必须遵守）


### 3.2 仓库源码目录

```
zacp/
├── AGENTS.md                 # 本文件：Agent 协作约定
├── README.md                 # 人类可读项目说明
├── Dockerfile                # 镜像构建（优先后端）
├── backend/                  # ★ 全部后端 Go 代码
│   ├── cmd/server/           # 进程入口，保持薄：组装依赖 + 启动
│   ├── configs/              # 仅样例：config.example.toml（入库）
│   ├── internal/             # 私有业务代码（禁止被外部模块 import）
│   │   ├── acp/
│   │   │   ├── client/       # ACP Client 连接封装（stdio/管道等）
│   │   │   ├── manager/      # 多 Agent / 多会话生命周期
│   │   │   └── providers/    # 各 Agent 启动参数与适配（pi、reasonix、grok…）
│   │   ├── api/
│   │   │   ├── handlers/     # HTTP Handler
│   │   │   ├── middleware/   # 中间件
│   │   │   └── router/       # 路由注册
│   │   ├── config/           # Viper 加载 $ZACP_DATA/config.toml → 强类型 Config
│   │   ├── store/            # GORM + SQLite：打开库、WAL、迁移、仓储
│   │   ├── model/            # DTO / 领域模型 / GORM model
│   │   ├── service/          # 业务编排（对接 manager / store 等）
│   │   └── ws/               # WebSocket：向浏览器推送 session/update
│   ├── pkg/                  # 可被多处复用的轻量工具库（谨慎使用）
│   ├── go.mod
│   └── go.sum
├── frontend/                 # ★ 全部前端（Vue 3 + Naive UI + Tailwind）
│   ├── src/
│   │   ├── api/              # REST 封装
│   │   ├── assets/
│   │   ├── components/       # 可复用 UI 组件
│   │   ├── composables/      # 可复用组合式函数（含 WS 等）
│   │   ├── pages/ 或 views/  # 页面
│   │   ├── stores/           # 状态（如 Pinia，脚手架时定）
│   │   ├── styles/           # 全局样式 / Tailwind 入口
│   │   ├── types/            # TS 类型（与后端 JSON 对齐）
│   │   ├── utils/            # 纯函数工具（优先复用）
│   │   ├── App.vue
│   │   └── main.ts
│   ├── public/
│   ├── package.json
│   └── vite.config.ts        # 或等价构建配置
├── scripts/                  # Shell 脚本（开发、构建、发布）
└── docs/                     # 设计文档、协议笔记
```

### 放置规则

| 内容 | 位置 |
|------|------|
| Go 业务与依赖 | **仅** `backend/` |
| Web UI 源码 | **仅** `frontend/`（Vue 3 + Naive UI + Tailwind） |
| 可执行脚本 | `scripts/` 或根目录（根目录仅放极少数全局脚本） |
| 配置样例（入库） | `backend/configs/config.example.toml` |
| 运行时配置 / 数据库 | **`$ZACP_DATA`**（默认 `~/.zacp`），**不入库** |

**禁止：**

- 在仓库根目录散落 Go 源码或 `go.mod`
- 在 `backend/` 外写业务 Go 包
- 把 `~/.zacp` 下的真实配置、数据库、密钥、前端构建产物提交进 Git

---

## 4. 后端分层约定

按依赖方向从上到下：

```
cmd/server  →  api (router/handlers)  →  service  →  acp/manager|client|providers
                     ↓                      ↓
                    ws                   config / store / model
```

- **`cmd/server`**：只做启动与依赖注入，不写复杂业务。
- **`api/handlers`**：解析请求、校验入参、调用 `service`，返回 JSON；不直接 `exec` Agent 进程。
- **`service`**：用例编排（创建会话、发 Prompt、取消、权限回传、读写历史）。
- **`acp/client`**：封装 `acp-go-sdk` 的连接与回调（实现 `acp.Client`：权限、读/写文件、终端等）。
- **`acp/manager`**：管理多个 provider 连接、session 映射、并发与清理。
- **`acp/providers`**：各工具的启动命令、参数、环境变量差异；新增 Agent 优先在此扩展，而不是改核心 manager 逻辑。
- **`store`**：GORM/SQLite 初始化、迁移、会话与消息等持久化。
- **`ws`**：把 ACP `SessionUpdate`（消息块、工具调用、计划等）转成前端可消费的事件。

新增 Agent 接入清单：

1. 在用户 `~/.zacp/config.toml` 的 `[[agents]]` 增加一项，并同步仓库 `config.example.toml`  
2. 在 `internal/acp/providers` 增加启动/连接适配（若通用 stdio 已够用可只配 command/args）  
3. 在 manager 按配置注册 / 启用  
4. 必要时扩展 API / 前端展示  

---

## 4.1 配置文件约定（TOML + Viper + `~/.zacp`）

### 文件位置

| 文件 | 是否入库 | 说明 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helloxz/zacp](https://github.com/helloxz/zacp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
