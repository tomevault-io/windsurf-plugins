---
trigger: always_on
description: > 本文档是 Tauri 桌面客户端迁移的**完整参考**，供 AI Agent 和开发者续接工作使用。
---

# [](https://)[](https://)[](https://)MCPHub Desktop (Tauri) — Agent 开发文档

> 本文档是 Tauri 桌面客户端迁移的**完整参考**，供 AI Agent 和开发者续接工作使用。
> 包含：原项目架构、桌面端架构、已完成内容、待办事项及所有关键技术细节。

> ⚠️ **核心约束（MUST FOLLOW）**：**禁止修改 `mcphub-origin/frontend/`、`mcphub-origin/src/` 等原始源文件**。
> 所有修改必须在 `frontend/`、`src-tauri/`、`locales/` 目录内进行。
> 做任何较大修改后，必须更新 agent.md 文档，用来记录。目的：为了方便后续维护和理解项目结构。

---

## 1. 项目概览[](https://)[](https://)

### 1.1 原项目（mcphub-origin — Node.js/Express + React/Vite）


| 属性     | 值                                                      |
| -------- | ------------------------------------------------------- |
| 包名     | `@samanhappy/mcphub`                                    |
| 技术栈   | Express.js + TypeScript ESM + React/Vite + Tailwind CSS |
| 前端     | `mcphub-origin/frontend/` (React + Vite)                |
| 认证     | JWT + bcrypt + Better-Auth（OAuth/OIDC）                |
| 数据存储 | JSON 文件 (`mcp_settings.json`) 或 PostgreSQL           |
| MCP 连接 | `src/services/mcpService.ts` 管理所有 MCP 服务端连接    |
| 路由     | `/mcp/{group                                            |
| i18n     | react-i18next，翻译文件在`locales/`                     |

### 1.2 桌面端项目（mcphub-desktop — Rust/Tauri 2 + 复用原 React 前端）


| 属性        | 值                                                        |
| ----------- | --------------------------------------------------------- |
| 位置        | 项目根目录                                                |
| Tauri 版本  | v2                                                        |
| Rust crate  | `src-tauri/`                                              |
| 前端        | `frontend/`（原 mcphub-origin/frontend 的副本，有改造）   |
| 数据存储    | SQLite（`$APPDATA/mcphub.db`，通过 sqlx 0.8）             |
| 认证        | jsonwebtoken 9 + bcrypt 0.15，密钥存 OS 钥匙串(keyring 3) |
| 异步运行时  | tokio 1 full                                              |
| HTTP 客户端 | reqwest 0.12 (rustls-tls + stream + json)                 |
| 应用标识    | `app.mcphub.desktop`                                      |

---

## 2. 桌面端架构

### 2.1 目录结构

```
mcphub-desktop/
├── frontend/                   # 原 mcphub-origin/frontend/ 的副本（有改造）
│   ├── src/
│   │   ├── pages/              # 页面组件（11个页面）
│   │   ├── components/         # 可复用 UI 组件
│   │   │   ├── layout/         # Header, Sidebar, Content
│   │   │   ├── ui/             # 通用 UI 组件
│   │   │   ├── icons/          # SVG 图标组件
│   │   │   ├── ServerCard.tsx   # ⚠️ 本地修改：移除 sponsor/wechat/discord
│   │   │   ├── ServerForm.tsx   # ⚠️ 本地修改：使用 hub-* 样式 + 保留 visibility/OAuth2
│   │   │   └── RuntimeVersionManager.tsx  # 🆕 桌面端新增：运行时版本管理
│   │   ├── utils/
│   │   │   ├── tauriClient.ts  # 🆕 isTauri() + invoke() 封装 + REST→invoke 路由映射
│   │   │   ├── fetchInterceptor.ts  # ⚠️ 修改：拦截请求转为 invoke()
│   │   │   └── runtime.ts      # 运行时配置
│   │   ├── contexts/
│   │   │   ├── AuthContext.tsx  # ⚠️ 修改：支持 skipAuth/guest 模式
│   │   │   └── ...
│   │   └── services/
│   │       └── configService.ts # ⚠️ 修改：getPublicConfig 使用 apiGet
│   ├── dist/                   # Vite 构建输出
│   └── package.json
├── locales/                    # i18n 翻译（en/zh/fr/tr）
│   ├── en.json                 # ⚠️ 本地修改：添加 runtime* 翻译
│   └── zh.json                 # ⚠️ 本地修改：添加 runtime* 翻译
├── mcphub-origin/              # git 子模块，仅作代码参考
├── src-tauri/                  # Rust 后端
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── migrations/
│   │   ├── 0001_initial.sql
│   │   ├── 0002_schema_fix.sql
│   │   ├── 0003_config_json.sql
│   │   ├── 0004_default_admin.sql
│   │   ├── 0005_default_skip_auth.sql  # 🆕 桌面端：默认开启免登录
│   │   └── 0006_openapi_column.sql    # 🆕 servers 表添加 openapi 列
│   └── src/
│       ├── main.rs
│       ├── lib.rs              # 应用核心：插件注册、setup hook、invoke_handler
│       ├── auth/
│       │   └── mod.rs          # JWT + bcrypt + guest token 签发
│       ├── db/
│       │   ├── mod.rs          # SQLite 连接池 + 初始化入口
│       │   └── migration.rs    # 🆕 版本化 DB 迁移管理模块
│       ├── models/
│       │   ├── server.rs       # ServerType, ServerConfig, ServerStatus, Tool
│       │   ├── user.rs         # User, UserRole(Admin|User|Guest), UserInfo
│       │   ├── group.rs
│       │   ├── config.rs
│       │   ├── auth.rs
│       │   ├── bearer_key.rs
│       │   └── log.rs
│       ├── mcp/
│       │   ├── client.rs       # McpTransport trait + McpClient
│       │   ├── stdio_transport.rs
│       │   ├── sse_transport.rs    # ⚠️ 本地修改：改进 SSE 事件解析
│       │   ├── http_transport.rs   # Streamable HTTP POST 传输
│       │   ├── openapi_transport.rs # 🆕 OpenAPI → MCP 传输（spawn rmcp-openapi）
│       │   └── pool.rs         # 全局连接池
│       ├── services/
│       │   ├── mod.rs
│       │   ├── mcp_manager.rs
│       │   ├── server_service.rs
│       │   ├── user_service.rs
│       │   ├── group_service.rs
│       │   ├── config_service.rs
│       │   ├── log_service.rs
│       │   ├── settings_import.rs
│       │   ├── bearer_key_service.rs
│       │   ├── http_server.rs      # 内置 HTTP 服务器（expose_http 模式）
│       │   ├── runtime_env.rs      # 🆕 运行时环境管理（Node.js/Python 版本隔离）
│       │   ├── server_tool_config_service.rs
│       │   └── market_service.rs
│       └── commands/
│           ├── mod.rs
│           ├── auth.rs         # login/logout/get_current_user/change_password
│           ├── servers.rs      # list/get/add/update/delete/toggle/reload
│           ├── groups.rs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skrstop/MCPHub-Desktop](https://github.com/skrstop/MCPHub-Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
