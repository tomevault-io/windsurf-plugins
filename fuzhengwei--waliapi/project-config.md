---
trigger: always_on
description: > 本文件面向 AI 编码代理，介绍 WaLiAPI 项目的架构、构建方式与开发约定。阅读本文件即可上手开发，无需先读其他文档。
---

# AGENTS.md

> 本文件面向 AI 编码代理，介绍 WaLiAPI 项目的架构、构建方式与开发约定。阅读本文件即可上手开发，无需先读其他文档。

## 项目概览

**WaLiAPI** 是一款本地运行的 LLM API 网关（当前版本 0.3.5，MIT 协议）。核心能力：

- **多协议网关**：下游可用 OpenAI Chat Completions、OpenAI Responses、Anthropic Messages 三种协议接入，出口统一转换后转发到上游供应商（OpenAI / Claude / DeepSeek / Gemini / 智谱 / 通义 / Moonshot / 豆包 / Ollama / 自定义）。
- **渠道调度**：优先级 + 权重负载均衡、多 API Key 负载、模型映射、自动故障切换。
- **安全审计**：风险扫描引擎（敏感信息、路径、Unicode 隐写等 25+ 内置规则），策略支持只审计 / 警告 / 脱敏 / 阻断。
- **知识库 RAG**：文档解析（tree-sitter 代码符号感知）→ 智能分块 → 向量化（复用渠道 Embedding）→ HNSW 向量索引 + SQLite FTS5 混合检索 → RAG 问答。
- **Wiki 知识引擎**：Markdown + frontmatter 页面管理、`[[wikilinks]]` 知识图谱、摄入管道。
- **MCP Server**：`/mcp` 端点（Streamable HTTP + SSE），对外暴露 29 个工具（知识库 13 + Wiki 16）。

同一套 Rust 代码库编译出两种产物：

- **桌面端**（默认 feature `desktop-ui`）：Tauri 2 窗口应用，macOS / Windows / Linux。
- **Headless 服务端**（`waliapi-web` 二进制，`--no-default-features --features embed-web`）：无窗口纯 HTTP 服务，用于 Docker / systemd 部署，内嵌 Web 管理面板。

## 技术栈

| 层 | 技术 |
|:---|:---|
| 前端 | React 19 + TypeScript ~5.8 + Vite 7 + Tailwind CSS 4 + React Router 7（无独立状态/服务端数据库，组件内 useState/useEffect + 自封装 runtime 层） |
| 后端 | Rust (edition 2021) + Tauri 2 + Axum 0.8 + sqlx 0.8 (SQLite) + reqwest 0.12 + tokio |
| 知识库 | tree-sitter（7 种语言）+ HNSW + FTS5 + bincode + pdf-extract |
| 包管理 | pnpm（workspace，含 `web/` 子包）+ cargo |
| 打包 | Tauri bundler（.dmg / .msi / .deb / .AppImage）、Docker 多阶段构建 |

## 仓库结构（实际目录）

```
├── src/                      # 前端主源码（桌面端直接用，Web 面板复用）
│   ├── pages/                # 9 个页面：Dashboard / Channels / AuthChannels / ApiKeys / Logs
│   │                         #   / KnowledgeBase / Usage / Settings / AppConfig
│   ├── components/           # ChannelForm、MappingSection、auth/、channel-form/、layout/ 等
│   ├── hooks/                # useModelMappings 等
│   ├── lib/                  # api.ts、runtime.ts（统一命令传输层，见下）、constants.ts
│   └── types/                # TypeScript 类型定义
├── web/                      # Web 管理面板构建（pnpm 子包 waliapi-web）
│   └── src/lib/              # tauri-shim.ts 等：把 @tauri-apps/* API 替换为 HTTP 实现
│                             # web/vite.config.ts 用 @app alias 复用 ../src 全部页面组件
├── src-tauri/                # Rust 后端
│   ├── src/
│   │   ├── main.rs           # 桌面端入口（default-run = "waliapi"）
│   │   ├── bin/waliapi-web.rs # headless 服务端入口
│   │   ├── lib.rs            # 库入口：AppState、系统托盘、服务启动
│   │   ├── web_server.rs     # headless 启动逻辑（数据目录解析、host/port）
│   │   ├── server/           # Axum HTTP 服务：router.rs、handlers.rs、admin*.rs（管理面）、
│   │   │                     #   event_bridge.rs（桌面 Webview / Web SSE 统一事件出口）
│   │   ├── core/             # 核心调度：proxy.rs、dispatcher.rs、route_plan.rs、
│   │   │                     #   plan_executor.rs、attempt.rs、stream_supervisor.rs、
│   │   │                     #   channel_identity.rs、feature_flags.rs、protocol_boundary.rs
│   │   ├── endpoint_executor/ # 端点执行器（请求驱动、SSE 处理、Token 用量估算）
│   │   ├── auth_provider/    # Auth 账号（Codex / Kimi / Antigravity OAuth 登录、Token 刷新、模型发现）
│   │   ├── protocol/         # 协议转换层：codec/（chat / messages / responses_codec /
│   │   │                     #   directions 双向转换）、sse_bridge.rs（字节级 SSE 重组，CJK 安全）
│   │   ├── adaptor/          # 上游渠道适配器：openai / claude / deepseek / gemini / custom
│   │   ├── security/         # 安全审计：scanner.rs、rules.rs、redact.rs
│   │   ├── services/         # 服务层：mod.rs 定义 Service trait + ServiceRegistry，
│   │   │                     #   knowledge/（RAG 全链路，含 ocr/ 扫描版 PDF VLM 识别）、wiki/、mcp/、channel_test.rs
│   │   ├── commands/         # Tauri Commands（channel / api_key / auth / log / settings / ...）
│   │   ├── db/               # Database 初始化、models.rs、repository.rs
│   │   ├── channel_presets.rs # 渠道预设注册表
│   │   └── settings_store.rs # 设置存储抽象（桌面 tauri-plugin-store / headless JSON 文件）
│   ├── migrations/           # SQL 迁移 001–027，启动时经 sqlx::migrate! 自动执行，迁移前自动备份 DB
│   ├── resources/pdfium/     # pdfium 动态库打包目录（VLM OCR 用，库文件不入库，见该目录 README）
│   └── tests/                # 集成测试（见「测试」一节）
├── deploy/                   # caddy/Caddyfile.example、systemd/（unit + env 示例）
├── scripts/                  # fetch-pdfium.sh（打包前下载 OCR 渲染依赖 pdfium 动态库）、Gitcode release 同步脚本
├── docs/                     # 设计文档（渠道协议重构、KB 升级、Wiki 架构等）
└── .github/workflows/        # 6 个发布 workflow（见「发布与部署」）
```

注意：README 中的「项目结构」一节可能滞后于代码（例如 `endpoint_executor/`、`auth_provider/` 现为顶层模块而非 `core/` 子模块），以实际目录为准。

## 构建与开发命令

包管理器用 **pnpm**（CI 使用 pnpm 11 + Node 22）。前端依赖安装：

```bash
pnpm install --frozen-lockfile
```

桌面端开发（前端 vite dev + Tauri 窗口）：

```bash
pnpm tauri dev        # 等价于 ./start.sh（内容为 npm run tauri dev）
```

前端构建（产物 `dist/`）：`pnpm build`（先 `tsc` 类型检查再 `vite build`）
Web 面板构建（产物 `web/dist/`）：`pnpm --filter waliapi-web build`

Rust 后端：

```bash
# 桌面端（默认 feature = desktop-ui，需要系统 GTK/WebKit 依赖）
cargo build --manifest-path src-tauri/Cargo.toml

# headless 服务端（关闭 desktop-ui，内嵌 Web 面板静态资源）
cargo build --release --manifest-path src-tauri/Cargo.toml \
  --bin waliapi-web --no-default-features --features embed-web
```

关键 Cargo feature（`src-tauri/Cargo.toml`）：

- `default = ["desktop-ui"]`：桌面专属能力（托盘、文件对话框、自启、OAuth 打开浏览器）。headless 构建必须 `--no-default-features`，否则会把 GTK/WebKit/DBus 链入二进制。
- `embed-web`：通过 rust-embed 把 `web/dist` 内嵌进 `waliapi-web` 二进制。
- release profile 开启 `lto = "fat"` + `codegen-units = 1`，请勿随意改动（用于剔除 headless 二进制中的桌面端代码）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fuzhengwei/WaLiAPI](https://github.com/fuzhengwei/WaLiAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
