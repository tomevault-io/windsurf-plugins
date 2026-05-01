---
trigger: always_on
description: This is a FastAPI + DeepAgents project (renamed to JellyfishBot) with the following structure after refactoring:
---

# JellyfishBot Project Rules

## Architecture
This is a FastAPI + DeepAgents project (renamed to JellyfishBot) with the following structure after refactoring:
Two-tier system: Admin Management + Consumer Consumption.

```
app/
  core/           - settings.py, security.py (auth), observability.py (langfuse)
                  - path_security.py (ensure_within / safe_join — unified path traversal prevention)
  storage/        - 可选 S3 文件存储后端 (STORAGE_BACKEND=local|s3 环境变量切换)
                  - config.py (S3Config, is_s3_mode)
                  - base.py (StorageService ABC)
                  - local.py (LocalStorageService — 封装 os.* 本地磁盘操作)
                  - s3.py (S3StorageService — boto3 S3 API)
                  - s3_backend.py (S3Backend — deepagents BackendProtocol for S3)
                  - __init__.py (工厂: get_storage_service, create_agent_backend, create_consumer_backend)
  schemas/        - requests.py (admin models), service.py (service/consumer models)
  routes/         - auth.py, conversations.py, chat.py, files.py, scripts.py,
                    models.py, settings_routes.py, batch.py
                  - services.py (admin service CRUD + key mgmt)
                  - consumer.py (consumer chat APIs: OpenAI compat + custom SSE)
                  - consumer_ui.py (standalone chat page at /s/{service_id})
  services/       - agent.py, tools.py, conversations.py, prompt.py,
                    subagents.py, ai_tools.py, script_runner.py
                  - _sandbox_wrapper.py (runtime file I/O sandbox for script execution)
                  - published.py (service CRUD, API key mgmt, consumer conversations)
                  - consumer_agent.py (consumer agent factory, with memory subagent)
                  - memory_tools.py (Memory subagent tools + soul config + short-term memory injection)
  voice/          - router.py (WebSocket S2S proxy)
  deps.py         - get_current_user (admin), get_service_context (consumer via sk-svc-)
  main.py         - FastAPI app assembly (64 routes) + startup/shutdown
frontend/                — Vite + React 18 + TypeScript + Ant Design 5
  index.html              — Vite entry point
  vite.config.ts          — dev server (port 3000), proxy /api → FastAPI :8000
  tsconfig.json           — strict TS, path alias @/* → src/*
  src/
    main.tsx              — React root mount
    App.tsx               — ConfigProvider (antd dark theme) + AuthProvider + Router
    router/index.tsx      — BrowserRouter: /login, /, /services, /scheduler, /wechat
    layouts/AppLayout.tsx — Sider (nav menu) + Content <Outlet/>
    pages/
      Login.tsx           — 品牌分栏登录/注册：左 40% 渐变品牌区（`/media_resources/jellyfishlogo.png` + 呼吸动画 + 像素点底纹），右 60% 表单区 `#1c1c27`；主色 Primary `#E89FD9`、Secondary `#8B7FD9`、Accent `#5FC9E6`；表单项聚焦描边与渐变按钮；样式以 inline + 组件内 `<style>`（keyframes）为主，窄屏约 900px 以下改为上下堆叠
      Chat/index.tsx      — conversation list + message area + SSE streaming
      AdminServices/index.tsx — Service 管理页：左 30% 服务列表（卡片 `#1c1c27`、圆角 8px、悬停边框 `#E89FD9`、选中左侧 3px `#E89FD9`、发布/草稿用绿/灰圆点）；右 70% 详情区 Tabs：Basic Info | API Keys | WeChat Channel | Test；API Keys 表斑马纹深色行；复制按钮用 Phosphor `Copy` + 点击高亮 Accent；图标 `@phosphor-icons/react`（Plus/Trash/PencilSimple/LinkSimple/ArrowsClockwise/GridFour）；品牌色与 `FLOAT_SHADOW`、`MODAL_RADIUS` 12px 与 Login/Chat 一致
      Scheduler/          — placeholder (migrating from scheduler.html)
      WeChat/             — placeholder (migrating from admin-wechat overlay)
    services/api.ts       — typed API client (port of legacy js/api.js)
    stores/authContext.tsx — React Context for auth state
    styles/
      global.css          — scrollbar, markdown, antd overrides
      theme.ts            — antd ThemeConfig matching original dark palette
    types/index.ts        — shared TS interfaces
  public/                 — static assets (legacy HTML kept for FastAPI template pages)
    service-chat.html     — consumer chat (served by FastAPI /s/{service_id})
    wechat-scan.html      — WeChat scan (served by FastAPI /wc/{service_id})
  server.js               — legacy Express proxy (kept, use `npm run legacy`)
  public/js/              — legacy vanilla JS (reference during migration)
  public/css/             — legacy CSS (reference during migration)
```

## Storage Backend (可选 S3)
- 通过环境变量 `STORAGE_BACKEND=local|s3` 切换，默认 `local`
- `local` 模式：所有文件操作使用本地磁盘（行为与原代码一致）
- `s3` 模式：文件存取走 S3 API（兼容 AWS S3、MinIO、R2、OSS 等）
- **工厂函数**：`get_storage_service()` 返回 StorageService 实例，`create_agent_backend()` 返回 deepagents BackendProtocol 实例
- **S3 键映射**：用户文件 `{prefix}/{user_id}/fs/{path}`，消费者生成文件 `{prefix}/{admin_id}/svc/{svc_id}/{conv_id}/gen/{path}`
- **媒体访问**：S3 模式下使用 presigned URL（302 重定向），local 模式使用 FileResponse
- **脚本执行**：S3 模式下临时下载脚本到本地执行，结果上传回 S3
- 新增依赖：boto3, aioboto3（仅 S3 模式使用）

## Two-Tier Design
- **Admin**: registers, manages docs/scripts/prompts, publishes Services
- **Service**: a published config selecting model + docs + scripts + capabilities
- **Consumer**: authenticates via per-service API key (`sk-svc-...`)
- **Isolation**: consumer generated content isolated per `conversation_id` under
  `users/{admin_id}/services/{service_id}/conversations/{conv_id}/generated/`
- **Consumer APIs**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiUshin/JellyfishBot](https://github.com/LiUshin/JellyfishBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
