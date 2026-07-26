---
trigger: always_on
description: > 本文件为 AI 编码助手提供项目上下文，遵循 Anthropic CLAUDE.md 最佳实践。
---

# CLAUDE.md — Banana Pro AI 项目指南

> 本文件为 AI 编码助手提供项目上下文，遵循 Anthropic CLAUDE.md 最佳实践。

## 项目概述

**大香蕉 AI (Banana Pro AI)** — 跨平台 AI 图片生成应用，支持 Gemini 和 OpenAI 标准接口。

- **版本**: v2.8.0
- **协议**: MIT
- **仓库**: ShellMonster/Nano_Banana_Pro_Web

三层架构：React 前端 → Tauri (Rust IPC 桥) → Go Sidecar 后端。桌面端通过 Tauri 打包，Web 端通过 Docker + Nginx 部署。

## 常用命令

```bash
# 后端
cd backend && go run cmd/server/main.go    # 启动后端 (默认 :8080)
curl http://localhost:8080/api/v1/health   # 标准健康检查接口
make build                                  # 编译
make run                                    # 运行

# 桌面端 (前端 + Tauri)
cd desktop && npm install && npm run tauri dev

# Web 前端 (独立版，非 Tauri)
cd frontend && npm install && npm run dev

# 打包发布
cd desktop && npm run tauri:build:local     # 本地构建桌面安装包（先生成 Go sidecar，跳过 updater 签名）

# Docker 部署 (Web 版)
docker compose -p banana-pro up -d

# 发布流程
git tag v2.8.0 && git push origin v2.8.0   # 触发 GitHub Actions 自动构建
# Release CI
cd desktop && npm ci                       # 发布构建使用 lockfile 严格安装依赖
```

## 项目结构

```
backend/                   # Go 后端 (Sidecar)
├── cmd/server/main.go     # 服务入口、路由注册、生命周期
├── internal/
│   ├── api/               # API Handlers (generate, folders, templates, export, history, config, images)
│   ├── provider/          # AI 提供商适配器
│   │   ├── gemini.go      # Gemini /v1beta 接口
│   │   ├── openai.go      # OpenAI /v1/chat/completions (多模态)
│   │   ├── openai_image.go # OpenAI /v1/images/generations (gpt-image-2)
│   │   └── model_resolver.go # 模型名称解析
│   ├── worker/pool.go     # Worker 池 (6 workers, 100 queue, panic recovery)
│   ├── model/             # 数据模型 (ProviderConfig, Task, Folder)
│   ├── storage/storage.go # 存储层 (本地文件系统 + 可选阿里云 OSS)
│   ├── config/config.go   # Viper 配置管理
│   ├── templates/store.go # 模板市场 (935+ 模板, 内嵌+远程+缓存)
│   ├── promptopt/service.go # 提示词优化 (text/json 模式, singleflight, 10min cache)
│   ├── diagnostic/        # 日志 & 错误分类 (20+ 类别)
│   ├── folder/            # 文件夹管理 (自动月分组 + 手动文件夹)
│   └── platform/runtime.go # 运行环境检测 (Tauri/Docker)

desktop/                   # Tauri 桌面端
├── src/
│   ├── components/        # 38 React 组件
│   ├── store/             # 8 个 Zustand Store (configStore migration v19)
│   ├── hooks/             # 6 个自定义 Hook
│   ├── services/          # 8 个 API 服务文件
│   ├── i18n/locales/      # 4 种语言 (zh-CN, en-US, ja-JP, ko-KR)
│   └── types/             # TypeScript 接口定义
├── src-tauri/
│   ├── Cargo.toml         # Rust 依赖
│   ├── tauri.conf.json    # Tauri 配置 (窗口、权限、Updater)
│   └── capabilities/      # Tauri 权限声明
└── package.json

frontend/                  # 独立 Web 前端 (非 Tauri，Docker 构建入口；版本跟随 desktop/package.json)
```

## 技术栈

| 层 | 技术 | 版本 |
|---|---|---|
| 前端 | React + TypeScript + Zustand | 18.3.1 |
| 桌面容器 | Tauri (Rust) | 2.0 |
| 后端 | Go + Gin | 1.21+ |
| 数据库 | SQLite (via mattn/go-sqlite3) | - |
| CI/CD | GitHub Actions | release.yml + pr-check.yml |
| 部署 | Docker multi-stage (Alpine + Nginx) | - |

## AI 提供商架构

项目支持 3 种 AI 提供商，通过 `ProviderType` 区分：

| Provider | API Endpoint | 用途 |
|---|---|---|
| `gemini` | `/v1beta/models/{model}:generateContent` | 文生图/图生图，支持 4K |
| `openai` | `/v1/chat/completions` (多模态) | 文生图/图生图/提示词优化 |
| `openai-image` | `/v1/images/generations` | 专用图片生成 (gpt-image-2) |

- 提供商配置热更新：存储在 SQLite，修改后立即生效，无需重启
- 默认模型：`gemini-2.0-flash-exp` (gemini), `gpt-4o` (openai), `gpt-image-2` (openai-image)

## 关键架构决策

1. **asset:// 协议**：桌面端注册原生资源协议，绕过 HTTP 栈加载本地图片，速度提升 300%
2. **Sidecar 模式**：Go 后端作为 Tauri sidecar 运行，Tauri 退出时自动清理进程
3. **Worker 池**：6 workers + 100-slot 队列，per-provider 超时，provider 调用在 worker goroutine 内执行并带 panic 自动恢复
4. **IPC 优化**：前后端只传文件路径，二进制数据通过 asset:// 协议直读
5. **Prompt 优化**：singleflight 去重 + 10min 缓存，支持 text/json 两种输出模式
6. **模板市场**：内嵌 JSON + 远程 GitHub Raw + 本地缓存三层策略，24h 自动刷新；桌面端模板网格使用 `react-window` 虚拟化渲染，避免 935+ 模板一次性挂载
7. **服务端连接超时**：Go HTTP Server 使用 5s ReadHeaderTimeout、30s ReadTimeout、120s IdleTimeout；WriteTimeout 保持 0，避免截断任务状态 SSE 长连接
8. **Docker Nginx 长连接代理**：Web 版 Nginx 在 `http` 上下文使用 `$http_upgrade` 到 `$connection_upgrade` 的 `map`，API 代理只设置一个 `Connection $connection_upgrade`，兼容普通 HTTP 请求和 WebSocket/SSE 升级场景
9. **桌面端语言包加载**：`desktop/src/i18n/index.ts` 只在启动资源中内置默认 `zh-CN`，`en-US` / `ja-JP` / `ko-KR` 通过动态 import 在切换语言前按需加载，避免启动时打包并解析全部 locale JSON
10. **Tauri asset/CSP 边界**：桌面端 `asset://` 仅允许应用数据、应用配置、应用缓存和临时目录范围，禁止回退到 `$HOME/**`；CSP 只放行本地应用资源、localhost API/SSE、`asset/blob/data/http(s)` 图片与必要的 Vite/Tauri 内联样式

## 代码风格

### Go 后端
- 标准 Go 项目布局 (`cmd/`, `internal/`)
- Gin 框架，中间件链式调用
- 错误处理：使用 `internal/diagnostic` 分类错误，返回结构化 JSON
- 配置：Viper 读取 `config.yaml`，环境变量覆盖
- 数据库：`database/sql` + `mattn/go-sqlite3`，不使用 ORM

### React 前端
- 函数组件 + Hooks，无 Class 组件
- Zustand 状态管理，`configStore` 管理 provider 配置和迁移
- API 调用集中在 `services/` 目录，组件通过 service 函数 + Zustand/本地 state 管理请求结果；当前不使用 React Query，重新引入前必须先确定查询缓存边界和迁移入口
- i18n 使用 `react-i18next`，翻译文件在 `i18n/locales/`
- Tailwind CSS 样式

### 通用
- 中文注释解释「为什么」而非「做什么」
- 提交信息格式：`feat:`, `fix:`, `chore:`, `docs:`
- 桌面发布版本号统一在 `Cargo.toml`, `tauri.conf.json`, `desktop/package.json`, `Cargo.lock`, `desktop/package-lock.json`
- Docker Web 版仍构建 `frontend/`，`frontend/package.json` 与 `frontend/package-lock.json` 的根版本必须跟随 `desktop/package.json`，避免 Web 镜像与当前应用版本元数据脱节

## 注意事项 & 易错点

1. **端口冲突**：Go sidecar 监听 `127.0.0.1:8080`，标准健康检查接口为 `GET /api/v1/health`；确保无其他进程占用。调试时检查 `lsof -i :8080`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShellMonster/Nano_Banana_Pro_Web](https://github.com/ShellMonster/Nano_Banana_Pro_Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
