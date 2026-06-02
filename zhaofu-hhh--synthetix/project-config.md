---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Synthetix 是一个 AI 视频剪辑平台，采用 **Tauri 2.0 桌面应用**架构。前端 Vue 3 嵌入 Tauri 窗口，后端 FastAPI 作为本地 API 服务（端口 9527）+ sidecar 打包。UI 采用**统一编辑器**：左侧工作区（剪辑方案/音频）+ 中间 AI 对话栏 + 右侧（素材库 + 视频预览），左右可折叠。顶部菜单栏提供文件操作、项目名称编辑和工具弹窗。

后端通过 **core-nexus-ai** 统一推理框架调用 LLM、TTS、ASR、多模态（Multimodal）等 AI 服务。

## 运行应用

```bash
# 桌面模式（一键启动后端 + Tauri 窗口）
python main.py

# 前端开发模式（可选，热更新，端口 9528，需同时运行后端）
cd synthetix-vue && npm run dev

# Tauri 生产构建
python build_backend.py    # PyInstaller 打包 Python 后端为 exe
npx tauri build            # 生成 .msi 和 .exe 安装包（需在项目根目录运行）
```

`python main.py` 启动流程：后台线程启动 uvicorn API → 前台运行 `npx tauri dev`（自动构建前端 + 打开桌面窗口）。若 npx 不可用则回退到纯 Web 模式。

- API 文档: http://127.0.0.1:9527/docs （Swagger UI）
- 前端开发模式: http://127.0.0.1:9528（需同时运行后端）

## 前端开发

```bash
cd synthetix-vue
npm run lint       # lint
npm run format     # 格式化
npm run build      # 构建到 dist/
```

## 数据库迁移

```bash
alembic revision --autogenerate -m "描述"   # 生成迁移文件
alembic upgrade head                         # 执行迁移
alembic current                              # 查看当前版本
```

添加新实体后，必须在 `alembic/env.py` 中导入才能被迁移识别。

## 测试

```bash
pytest tests/unit/ -v                           # 运行单元测试
pytest tests/unit/test_xxx.py -v                # 运行单个测试
pytest tests/unit/ --cov=src --cov-report=html  # 覆盖率报告
```

## 架构

```
src/
├── agent/                        # 对话式剪辑 Agent
│   ├── react_agent.py            # ReAct Agent（TAOR 循环 + SSE 流式 + 深度研究）
│   ├── tool_registry.py          # @registry.register() 注册工具（含 Pydantic 校验、Hook、权限）
│   ├── session_manager.py        # 会话管理（内存缓存 + DB 双写）
│   ├── mcp_client.py             # MCP 协议客户端（动态接入外部工具服务器）
│   ├── extension_loader.py       # 扩展/插件加载器（扫描 src/extensions/ 目录）
│   ├── project_memory.py         # 项目级用户偏好记忆
│   ├── knowledge_base.py         # BM25 知识库（轻量 RAG）
│   └── multi_agent.py            # 多 Agent 协作（Planner→Executor→Reviewer）
│
├── extensions/                   # 扩展/插件目录（每个子目录一个扩展，含 manifest.json）
│
├── scripts/                      # 工具脚本（migrate_imports, update_imports）
│
├── domain/entities/              # SQLAlchemy 实体（12 个：VideoSource, AudioSource, VideoProject, ClipPlanItem, BGMItem, DialogSession, ComicProject, ComicSeries, ProjectTempFile, VideoShot, ConfigStore）
├── application/services/         # 业务服务
├── interfaces/api/               # FastAPI 路由
├── shared/                       # 常量、模型、工具函数
└── infrastructure/               # 数据库会话、Alembic、Repository

synthetix-vue/                    # 前端 Vue 3 + Vite + Pinia + Element Plus
synthetix-tauri/                  # Tauri 2.0 桌面应用（Rust）
├── src/main.rs, lib.rs           # Rust 入口（sidecar 启动）
├── tauri.conf.json               # Tauri 配置（窗口、构建、sidecar）
├── capabilities/                 # 权限声明
├── icons/                        # 应用图标
└── binaries/                     # sidecar 放置目录（PyInstaller 输出）

config/                           # 分层配置（default.json + DB config_store 表）
```

## 核心约定

### 命名转换
后端 snake_case → `success_response(to_camel=True)` 自动转 camelCase → 前端使用 camelCase。
前端 `debounceSave` 发送 snake_case key（`material_ids`, `target_duration` 等），后端 Pydantic 模型匹配 snake_case。

### 路由顺序
FastAPI 静态路由必须在动态路由（`/{id}`）之前定义，否则 `"bgm"` 等会被当作 `project_id`。

### 项目中心化
统一编辑器基于项目：
- 进入页面显示欢迎视图 → 新建/选择项目 → 加载编辑器
- 顶部菜单栏"文件"菜单可新建、切换、保存、导出项目
- 从项目列表点"打开" → 路由到 `/editor?projectId=X`
- 项目名称唯一，创建/修改时校验重复
- 前端 `watch` 各字段变化 → `debounceSave` (300ms, 每字段独立 timer) → `projectApi.update`

### 前端请求层
- `API_HOST` 单一来源：`synthetix-vue/src/utils/request.js`（读 `VITE_API_BASE_URL` 环境变量或默认 `http://127.0.0.1:9527`）
- `src/api/request.js`（axios）：所有 API 模块使用，拦截器自动提取 `data.data`，`success: false` 自动 reject 并弹 ElMessage
- `src/api/modules/` 下的 API 模块统一导出，通过 `src/api/modules/index.js` 聚合
- **SSE/流式请求**（`/api/agent/chat/stream`、`/api/nexus/llm/stream`）必须用原始 `fetch` + `ReadableStream`，不能用 axios
- `src/utils/request.js`（fetch）：仅导出 `assetUrl`、`API_HOST`，供 SSE 流式和外部 URL 使用
- 新增 API 方法：在 `src/api/modules/` 对应模块中添加，自动获得 axios 拦截器处理（`data.data` 提取、错误处理）

## Tauri 桌面应用

### 开发模式
`python main.py` 一键启动：uvicorn 后台线程 + `npx tauri dev` 前台进程。

### Sidecar（Python 后端打包）
- `build_backend.py` 用 PyInstaller 将 `main.py` 打包为 `synthetix-tauri/binaries/backend-x86_64-pc-windows-msvc.exe`
- Tauri 启动时通过 `tauri_plugin_shell` 拉起 sidecar
- 开发阶段 sidecar 不可用时跳过，依赖后台线程的 uvicorn

### 关键文件
- `synthetix-tauri/tauri.conf.json` — `frontendDist: ../synthetix-vue/dist`，`beforeDevCommand` 自动构建前端
- `synthetix-tauri/src/lib.rs` — sidecar 启动逻辑，失败不阻塞
- `synthetix-tauri/capabilities/default.json` — shell/process 权限

### 前置依赖
- Rust (rustup) — `winget install Rustlang.Rustup`
- Node.js — 已有
- Cargo PATH — `main.py` 自动添加 `~/.cargo/bin`

## 对话式 Agent 架构

### ReAct Agent（`react_agent.py`）

采用 **"笨引擎 + 聪明模型"** 架构：运行时不含业务逻辑，所有智能决策由 LLM 完成。

**TAOR 循环**: Think → Act → Observe → Repeat
```
用户输入 → LLM 思考 → 调用工具（或直接回复） → 观察工具结果 → 继续思考 → ... → 最终回复
```

**工具调用格式**（LLM 输出，正则解析）：
```
<tool_call name="tool_name">
{"param": value}
</tool_call}
```

**系统提示词注入链**（`_build_messages()` 按顺序拼接）：
1. 基础系统提示词（角色 + 工具描述 + 规则）
2. 项目偏好记忆（`project_memory.py` 提取的历史偏好）
3. 扩展提示词（`extension_loader.py` 加载的扩展声明）
4. MCP 外部工具描述（`mcp_client.py` 发现的远程工具）

**工具描述两模式策略**（`_build_messages()` 中根据 `active_extensions` 选择）：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhaofu-hhh/synthetix](https://github.com/zhaofu-hhh/synthetix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
