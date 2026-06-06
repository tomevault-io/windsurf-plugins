---
trigger: always_on
description: - **回答用户必须使用中文**：所有回复、任务清单及计划文件，均须使用中文
---

## 语言规范
- **回答用户必须使用中文**：所有回复、任务清单及计划文件，均须使用中文

## 项目概述

ArcReel 是一个 AI 视频生成平台，将小说转化为短视频。三层架构：

```
frontend/ (React SPA)  →  server/ (FastAPI)  →  lib/ (核心库)
  React 19 + Tailwind       路由分发 + SSE
  wouter 路由               agent_runtime/
  zustand 状态管理          (Claude Agent SDK)
```

## 开发命令

```bash
# 后端
# 启动开发服务器（必须用 --reload-dir 限定监视目录，否则 watchfiles 会扫描
# node_modules / .venv / .git / .worktrees 等十几万个文件，单核 CPU 50%+）
uv run uvicorn server.app:app --reload --reload-dir server --reload-dir lib --port 1241

uv run python -m pytest                              # 测试（-v 单文件 / -k 关键字 / --cov 覆盖率）
uv run ruff check . && uv run ruff format .          # lint + format
uv run basedpyright                                  # 类型检查（CI 强制 0 error）
uv sync                                              # 安装依赖
uv run alembic upgrade head                          # 数据库迁移
uv run alembic revision --autogenerate -m "desc"     # 生成迁移

# 前端，先 cd frontend
pnpm lint        # ESLint，CI frontend-tests 第一段，含 jsx-a11y 规则
pnpm check       # typecheck + vitest
pnpm build       # 生产构建，含 typecheck
# CI 等价：pnpm lint && pnpm check，push 前两条都要绿
```

## 架构要点

### 后端 API 路由

所有 API 在 `/api/v1` 下，路由定义在 `server/routers/`：
- `projects.py` — 项目 CRUD、概述生成
- `generate.py` — 分镜/视频/角色/场景/道具生成（入队到任务队列）
- `assistant.py` — Claude Agent SDK 会话管理（SSE 流式）
- `agent_chat.py` — 智能体对话交互
- `tasks.py` — 任务队列状态（SSE 流式）
- `project_events.py` — 项目事件 SSE 推送
- `files.py` — 文件上传与静态资源
- `versions.py` — 资源版本历史与回滚
- `characters.py` / `scenes.py` / `props.py` — 项目级资产 CRUD（**由 `_asset_router_factory.build_asset_router()` 统一生成**，按 `lib/asset_types.ASSET_SPECS` 驱动；新增资产类型时只需在 spec 注册）
- `assets.py` — 全局资产库（跨项目复用的 character/scene/prop，DB 持久化于 `assets` 表）
- `reference_videos.py` — 参考视频→视频生成（按镜头解析 + 入队）
- `usage.py` — API 用量统计
- `cost_estimation.py` — 费用预估（项目/单集/单镜头）
- `grids.py` — 宫格图生成、列表、详情、重新生成
- `auth.py` / `api_keys.py` — 认证与 API 密钥管理
- `system_config.py` — 系统配置
- `providers.py` — 预置供应商配置管理（列表、读写、连接测试）
- `custom_providers.py` — 自定义供应商 CRUD、模型管理与发现、连接测试

### server/services/ — 业务服务层

- `generation_tasks.py` — 分镜/视频/角色/场景/道具生成任务编排
- `reference_video_tasks.py` — 参考视频→视频生成任务编排
- `project_archive.py` — 项目导出（ZIP 打包）
- `project_cover.py` — 项目封面生成
- `project_events.py` — 项目变更事件发布
- `jianying_draft_service.py` — 剪映草稿导出
- `cost_estimation.py` — 费用预估计算与实际费用汇总
- `resolution_resolver.py` — 视频分辨率解析（按 provider 能力适配）

### lib/ 核心模块

- **{gemini,ark,grok,openai,vidu}_shared** + **httpx_shared** — 各供应商 SDK 工厂与共享工具
- **image_backends/** / **video_backends/** / **text_backends/** — 多供应商媒体后端，Registry + Factory 模式（vidu 仅 image/video；video 额外有 newapi 中转）
- **custom_provider/** — 自定义供应商支持：后端包装、模型发现、工厂创建（OpenAI/Google 兼容）
- **MediaGenerator** (`media_generator.py`) — 组合后端 + VersionManager + UsageTracker
- **GenerationQueue** (`generation_queue.py`) — 异步任务队列，SQLAlchemy ORM 后端，lease-based 并发控制
- **GenerationWorker** (`generation_worker.py`) — 后台 Worker，分 image/video 两条并发通道
- **ProjectManager** (`project_manager.py`) — 项目文件系统操作和数据管理
- **StatusCalculator** (`status_calculator.py`) — 读时计算状态字段，不存储冗余状态
- **UsageTracker** (`usage_tracker.py`) / **CostCalculator** (`cost_calculator.py`) — 用量追踪与费用计算
- **TextGenerator** (`text_generator.py`) / **ScriptGenerator** (`script_generator.py`) — 文本与剧本生成
- **asset_types.py** — character/scene/prop 三类资产的统一 spec（`ASSET_SPECS`），驱动路由工厂、bucket key、sheet 字段、PATCH 白名单
- **source_loader/** — 小说源文件导入（txt/docx/epub/pdf），统一 `loader` 接口
- **reference_video/** — 参考视频→视频：`shot_parser` 按镜头解析 prompt + `limits` 容量约束
- **grid/** — 宫格图系统：布局计算（grid_4/6/9）、prompt 构建、切割
- **agent_session_store/** — Claude Agent SDK transcript 入库镜像（store + import_local）
- **retry** (`retry.py`) — 通用指数退避重试装饰器，各供应商后端复用

### lib/config/ — 供应商配置系统

ConfigService（`service.py`）→ Repository（持久化 + 密钥脱敏）→ Resolver（解析）。`registry.py` 维护预置供应商注册表（PROVIDER_REGISTRY）。

### lib/db/ — SQLAlchemy Async ORM 层

- `engine.py` — 异步引擎 + session factory（`DATABASE_URL` 默认 `sqlite+aiosqlite`）
- `models/` — ORM 模型：Task / ApiCall / ApiKey / AgentSession（`session.py`）/ Config / Credential / User / CustomProvider（含模型子表）/ **Asset**（全局资产库）
- `repositories/` — 异步 Repository：Task / Usage / Session / ApiKey / Credential（多 API Key + 活跃切换）/ CustomProvider / **Asset**

数据库：开发 SQLite（`projects/.arcreel.db`），生产 PostgreSQL（`asyncpg`）

### Agent Runtime（Claude Agent SDK 集成）

`server/agent_runtime/` 封装 Claude Agent SDK：
- `AssistantService` (`service.py`) — 编排 Claude SDK 会话
- `SessionManager` — 会话生命周期 + SSE 订阅者模式
- `SessionActor` (`session_actor.py`) — 每会话一个专属 asyncio task，串行化所有 ClaudeSDKClient 调用（spec: `docs/superpowers/specs/2026-04-13-session-actor-design.md`）
- `SessionStore` (`session_store.py`) — 会话元数据 + transcript DB 镜像（受 `ARCREEL_SDK_SESSION_STORE` 环境变量控制：`db`/`off`，off 时回退到 SDK 自带的 jsonl 路径）
- `StreamProjector` — 从流式事件构建实时助手回复
- `sdk_transcript_adapter` / `turn_schema` — transcript 读取与 Turn 规范化（用于历史回放）
- `sdk_tools/` — SDK 进程内 MCP 工具（enqueue_assets/grid/storyboards/videos + text_generation），供 Skill 调用，由 agent profile manifest 注入

### lib/i18n/ — 国际化

后端翻译层，支持 `zh`/`en`/`vi` 三种语言。`{zh,en,vi}/` 各文件按命名空间拆分：`errors`（错误与校验）、`providers`（供应商名称/描述）、`assets`（资产相关消息）、`emails`（邮件模板）、`system`（系统消息）、`templates`（模板消息）。
- `Translator` 类型 = `Annotated[Callable[..., str], Depends(get_translator)]`，从 `Accept-Language` 解析语言

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javpower/-ArcReel](https://github.com/javpower/-ArcReel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
