---
trigger: always_on
description: > 本文档面向 AI Agent / 开发者，约定项目目标、架构边界与编码规范。
---

# i-code — Agent 工作指南

> 本文档面向 AI Agent / 开发者，约定项目目标、架构边界与编码规范。  
> 详细设计见 `docs/`，本文只保留**每次任务都需要遵守**的关键约束。

---

## 1. 项目定位

**i-code** 是基于 **Tauri 2.x** 的本地桌面应用，用于：

1. **管理 AI Gateway 供应商**：集中维护多 LLM 供应商（OpenAI、Anthropic、Gemini、OpenRouter 等），支持多协议与认证。
2. **提供本地 API Gateway**：统一暴露模型 ID 为 `{provider_slug}/{model_id}`，本地监听并代理请求。
3. **管理 CLI 配置**：为 Claude Code、Codex、Gemini CLI 等维护配置档案，支持直连或路由到本地 Gateway。

版本：`0.3.9`  
包管理：`pnpm@11`  
数据库：本地 SQLite（`i-code.db`）  
默认网关：`127.0.0.1:54321`

### 核心业务规则

| 规则 | 说明 |
|------|------|
| 模型路由 ID | 对外 = `{provider_slug}/{model_id}`；网关拆分后路由到真实供应商 |
| CLI 路由模式 | `base_url` 指向本地网关，`model` 字段保留前缀 |
| 敏感数据 | API Key / Token **禁止明文落库**；配置中仅存 `$SECRET:{uuid}$` |
| Secret 边界 | 加解密**仅在 Rust 后端**；前端只传明文一次，不缓存 |

---

## 2. 技术栈（勿随意更换）

| 层级 | 技术 |
|------|------|
| 桌面 | Tauri 2.x（Rust + WebView） |
| 前端 | React 19 + TypeScript 5（严格模式） |
| 路由 | TanStack Router（文件系统路由，`src/routes/`） |
| UI | shadcn/ui + Tailwind CSS + Font Awesome |
| 状态 | Zustand（前端）+ Tauri State（后端） |
| 表单 | react-hook-form + zod |
| 国际化 | i18next（`zh-CN` / `en`） |
| 后端 HTTP 网关 | axum |
| DB | rusqlite + r2d2 |
| 加密 | AES-GCM（本地模式；密钥链模式待迭代） |
| 类型同步 | ts-rs（Rust → TypeScript） |

**不要**引入 React Query / SWR（数据走 Tauri Command 一次性调用 + Event 推送）。  
**不要**在新代码中使用 `lucide-react`（图标统一 Font Awesome）。

---

## 3. 架构与模块边界

### 3.1 依赖方向

```
core/shared（零业务依赖）
    ↑
theme / i18n / secret / db / balance / logger / backup
    ↑
settings / ai-gateway / cli-management / gateway-runtime / virtual-provider
    ↑
frontend (components / routes / hooks)
```

### 3.2 模块清单

| 模块 | 职责 | 前端 | 后端 |
|------|------|------|------|
| `ai-gateway` | 供应商、模型、认证、代理 | ✅ | ✅ |
| `gateway-runtime` | 本地 HTTP 网关生命周期与转发 | ✅ 状态 | ✅ |
| `virtual-provider` | 虚拟供应商、故障转移路由 | ✅ | ⚠️ 部分 |
| `script-template` | 额度监控 Rhai 脚本模板 CRUD / 试运行 | ✅ | ✅ |
| `cli-management` | CLI 档案、绑定、模型映射 | ⚠️ 类型/骨架 | ⚠️ 骨架 |
| `secret` | 敏感数据加密与引用 | 仅输入组件 | ✅ AES 本地 |
| `settings` | 主题/语言/网关地址等 | ✅ | ✅ |
| `balance` | 额度查询（含自定义脚本） | ✅ | ✅ |
| `logger` | 运行时日志 | ✅ | ✅ |
| `call-records` | 模型调用统计 | ✅ | ✅ |
| `backup` | DB 备份/恢复、WebDAV | ✅ | ✅ |
| `theme` / `i18n` | 主题与双语 | ✅ 仅前端 | — |

前后端模块**同名对应**：`src/modules/{name}/` ↔ `src-tauri/src/modules/{name}/`。

### 3.3 分层规则（必须遵守）

**前端模块：**
- 仅 `types.ts` + `ui/`（+ 必要时 hooks 放 `src/hooks/`）
- **无** Repository / Service 层
- 数据访问一律 `invoke` → 后端 Command（用 `use-command` 封装）

**后端模块：**
```
commands.rs  → 参数校验、调 Service、错误转换
service.rs   → 业务逻辑与编排（可调其他模块 Service）
repository.rs → 仅 SQL / 数据访问（禁止调 Service、禁止发事件）
types.rs     → DTO / 领域类型
```

- Service **禁止**直接访问其他模块的 Repository
- Repository **禁止**调用 Service
- 跨模块只读数据：通过对方 Service 暴露的接口

### 3.4 详细文档（按需查阅，勿全文复制进回复）

| 文档 | 内容 |
|------|------|
| `docs/development.md` | 完整架构、模块设计、UI 规范、Commands 清单 |
| `docs/database.md` | Schema、JSON 字段约定、业务流程 |
| `docs/gateway-runtime.md` | 网关运行时生命周期、路由转发、Provider 适配器 |
| `docs/chat-module.md` | 聊天界面布局、发送/流式/错误气泡、JSONL 存储与 Command |
| `docs/events.md` | 前后端事件总线（含 `chat:stream-*`） |
| `docs/proxy.md` | 网络代理两层配置体系、核心函数、网络路径与日志约定 |
| `docs/log-framework.md` | 日志框架两套机制约定、网关日志格式、级别控制 |
| `docs/error-handling.md` | 错误处理体系、`IcodeError` 结构、前后端转换规范 |
| `docs/cli-management.md` | CLI 档案管理、绑定、模型映射设计 |
| `docs/proposals/balance-script-templates.md` | 额度监控脚本模板（Rhai）CRUD、运行时、编辑体验 |
| `docs/proposals/` | 待实现/演进提案 |
| `docs/fix-bug.md` | 已知问题与修复记录 |
| `docs/mimo-free-reference.md` | 小米 MiMo 免费通道（`mimo-auto`）参考项目协议调研：bootstrap/JWT、特殊请求头、guard prompt、限流 |
| `docs/kiro-reference.md` | Kiro IDE 通道参考项目协议调研：Google/GitHub 社交 OAuth2.0 授权（PKCE + `kiro://` deep link）与 token 刷新、`generateAssistantResponse` 模型调用与 EventStream、`ListAvailableModels` 模型列表、`getUsageLimits` 余额 |
| `参考项目/vscode-unify-chat-provider-7.12.3/` | 类型与 well-known 数据参考源 |

---

## 4. 目录速查

```
i-code/
├── docs/                      # 设计与提案
├── src/                       # 前端 React
│   ├── core/                  # types / errors / events / utils / constants
│   ├── hooks/                 # 跨模块 hooks（use-command 等）
│   ├── components/
│   │   ├── ui/                # shadcn + 全局通用组件（禁止业务类型）
│   │   ├── layout/            # 侧栏布局等
│   │   └── preview/           # /preview 演示用，无业务逻辑
│   ├── modules/{domain}/      # types.ts + ui/
│   ├── routes/                # TanStack 文件路由
│   ├── main.tsx
│   └── index.css
├── src-tauri/                 # 后端 Rust
│   ├── src/
│   │   ├── main.rs            # 入口、托盘、迷你窗、command 注册
│   │   ├── error.rs
│   │   ├── db/                # 连接、schema、migrations/
│   │   └── modules/           # 与前端 modules 一一对应
│   ├── data/                  # builtin-models/providers JSON
│   ├── tauri.conf.json
│   └── Cargo.toml
├── scripts/                   # 内置数据转换等
└── package.json
```

### 主要前端路由

| 路径 | 说明 |
|------|------|
| `/` | 仪表盘 |
| `/gateways`、`/gateways/providers`、`/gateways/models`、`/gateways/settings` | AI Gateway |
| `/cli` | CLI 管理 |
| `/logs` | 日志 |
| `/settings` | 设置 |
| `/preview` | 组件预览（开发用） |
| `/mini-panel` | 迷你悬浮窗（无 TitleBar / 无侧栏） |

---

## 5. GUI 与 UI 硬约束

### 5.1 窗口

主窗口固定设计尺寸 **900×700**（`tauri.conf.json`，`decorations: false`）：

- 布局优先**紧凑**，避免大片空白
- 理论左右布局在此宽度下可能变成上下布局——做响应式时用小断点验证
- 优先使用 shadcn 组件库已有组件

### 5.2 主题与样式

- 主题：`light` / `dark` / `claude-light` / `claude-dark` / `deepseek-light` / `deepseek-dark` / `nvidia-light` / `nvidia-dark`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xucux/i-code](https://github.com/xucux/i-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
