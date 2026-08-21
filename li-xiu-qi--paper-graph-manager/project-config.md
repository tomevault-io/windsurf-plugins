---
trigger: always_on
description: > 面向 AI 编码代理的项目速查手册。如果你需要更多上下文，请阅读 `README.md` 和 `docs/technical-decisions.md`。
---

# Paper Graph Manager — AI Coding Agent Guide

> 面向 AI 编码代理的项目速查手册。如果你需要更多上下文，请阅读 `README.md` 和 `docs/technical-decisions.md`。

---

## 1. 项目概述

Paper Graph Manager 是一套面向个人研究的论文管理工具，覆盖从论文发现到知识沉淀的完整工作流：arXiv 搜索与入库、PDF 上传、智能标注、知识图谱（团队视图 / 论文视图）、智能聊天、Markdown 笔记管理。

- 当前版本：v0.4.0（MVP 开发中）
- 定位：本地优先、单用户、轻量运维
- 许可证：Apache License 2.0

---

## 2. 技术栈

| 层 | 技术 |
|---|---|
| 后端 | FastAPI + Python 3.12 + Uvicorn |
| 前端 | React 19 + Vite 7 + TypeScript |
| UI | shadcn/ui + Tailwind CSS 3 + Radix UI |
| 数据库 | SQLite（单文件 `data/papers.db`） |
| 图谱 | NetworkX（计算）+ PixiJS + d3-force（渲染） |
| AI | OpenAI 兼容接口（通过 Kimi Agent SDK 调用） |
| PDF | PyMuPDF（fitz） |
| 测试 | pytest（后端）+ Vitest（前端） |

---

## 3. 目录结构与核心模块

```
paper-graph-manager/
├── backend/
│   ├── main.py                    # FastAPI 入口，所有 API 路由
│   ├── paper_graph/
│   │   ├── database.py            # SQLite schema、CRUD、聊天会话/消息
│   │   ├── ingest.py              # arXiv 搜索/入库、本地 PDF 入库、PDF 元数据提取
│   │   ├── annotate.py            # LLM 智能标注（核心贡献 + 团队识别）
│   │   ├── graph.py               # NetworkX 图谱构建 + PyVis HTML 导出
│   │   ├── notes.py               # Markdown 笔记 CRUD、frontmatter 解析
│   │   ├── export.py              # Markdown 格式图谱导出
│   │   ├── chat_agent.py          # Kimi Agent SDK Session 运行器（同步/流式）
│   │   ├── chat_tools.py          # 聊天工具实现（schema + 执行函数）
│   │   └── kimi_tools.py          # CallableTool2 包装器，供 agent.yaml 使用
│   ├── tests/
│   │   ├── conftest.py            # pytest fixtures：tmp_db、client、sample_paper
│   │   ├── test_main.py           # FastAPI TestClient 集成测试
│   │   ├── test_ingest.py         # 入库/解析/搜索单元测试
│   │   ├── test_annotate.py       # 标注逻辑 + LLM 降级测试
│   │   ├── test_graph.py          # 图谱构建 + 可视化导出测试
│   │   ├── test_notes.py          # 笔记系统 + frontmatter 测试
│   │   └── test_export.py         # Markdown 导出测试
│   ├── agent.yaml                 # Kimi Agent SDK 工具声明
│   ├── requirements.txt           # Python 依赖
│   ├── .env.example               # 环境变量模板
│   └── .env                       # 本地 LLM 配置（不提交）
├── frontend/
│   ├── src/
│   │   ├── App.tsx                # React Router 路由配置
│   │   ├── main.tsx               # 入口
│   │   ├── pages/                 # Dashboard、Papers、Graph、Chat、Notes、PaperDetail
│   │   ├── components/            # Layout、PixiGraph、NoteEditor、FileTree、UI 组件
│   │   ├── services/api.ts        # 所有 API 调用（含 SSE 流式聊天）
│   │   ├── types/index.ts         # TypeScript 类型定义
│   │   └── lib/utils.ts           # 工具函数（cn 等）
│   ├── package.json
│   ├── vite.config.ts             # Vite 配置 + Vitest 配置 + API 代理
│   ├── tsconfig.json
│   └── tailwind.config.js
├── data/
│   ├── papers.db                  # SQLite 数据库（运行时生成）
│   ├── pdfs/                      # PDF 原文存储
│   └── notes/                     # Markdown 笔记存储
├── docs/
│   └── technical-decisions.md     # 技术选型决策记录
├── pyproject.toml                 # Python 项目配置 + pytest 配置
├── test_e2e.py                    # 端到端流程验证脚本
└── README.md
```

---

## 4. 构建与运行命令

### 环境要求

- Python 3.12+
- Node.js 24+（已验证 v24.16.0）
- OpenAI 兼容 API Key（用于智能标注和聊天 Agent）

### 后端

```bash
cd backend
.venv\Scripts\activate          # Windows；Linux/macOS 用 source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

- 默认地址：`http://localhost:8000`
- 健康检查：`GET /api/health` 或 `GET /health`
- 后端日志：`backend/server.log`（超过 5MB 启动时自动清空）

### 前端

```bash
cd frontend
npm install
npm run dev -- --port 3000      # 或默认 5173
```

- 默认地址：`http://localhost:3000`（或 `http://localhost:5173`）
- Vite 开发服务器自动代理 `/api` 到 `http://localhost:8000`
- CORS 白名单：`http://localhost:3000`、`http://localhost:5173`

### E2E 验证

```bash
python test_e2e.py
```

该脚本会创建临时数据库，验证数据库初始化、arXiv 搜索/入库、图谱构建、笔记系统全流程。

---

## 5. 测试策略

### 后端测试

```bash
cd backend
pytest                           # 运行全部测试
pytest -k test_graph             # 运行指定模块
```

- **测试发现**：`pyproject.toml` 配置 `testpaths = ["backend/tests"]`、`pythonpath = ["backend"]`
- **异步模式**：`asyncio_mode = "auto"`
- **核心模式**：
  - `conftest.py` 提供 `tmp_db`（临时 SQLite）、`sample_paper`、`client`（TestClient）
  - `test_main.py` 通过 monkeypatch 替换 `DB_PATH` 实现隔离
  - 大量使用 `unittest.mock` 模拟 LLM 响应和 arXiv 客户端
- **覆盖率配置**：`pyproject.toml` 中 `[tool.coverage.run]` 指向 `backend/paper_graph`

### 前端测试

```bash
cd frontend
npm run test:run                 # 非交互运行
npm run test                     # 交互模式（watch）
```

- **测试环境**：Vitest + Node 环境（非 jsdom）
- **配置位置**：`vite.config.ts` 中的 `test` 字段
- **测试范围**：
  - `services/api.test.ts`：API 函数成功/失败路径
  - `pages.test.tsx`：页面组件可渲染性验证
  - **不引入组件交互测试**（项目规模小，手动测试成本低）

### E2E

- `test_e2e.py`：Python 集成脚本，覆盖数据库初始化、入库、标注、图谱、笔记全流程
- **不引入 Playwright/Cypress**：个人项目迭代快，E2E 维护成本高

---

## 6. 数据存储约定

### SQLite 数据库

- 路径：`data/papers.db`（运行时自动创建）
- 核心表：`papers`、`authors`、`institutions`、`paper_authors`、`paper_institutions`、`teams`、`team_members`、`paper_teams`、`chat_sessions`、`chat_messages`
- `init_db()` 会自动创建表并兼容旧库（如动态添加 `md_path` 列）

### 论文 ID 格式

- arXiv 入库：`arxiv_<arxiv_id>`（如 `arxiv_2402.09199`）
- 本地 PDF：`local_<file_hash[:12]>`
- 统一通过 `paper_id` 参数引用

### 文件存储

- PDF：`data/pdfs/{paper_id}.pdf`
- 笔记：`data/notes/{paper_id}.md`
- 笔记 frontmatter 格式：YAML-like，字段包括 `title`、`authors`、`published`、`categories`、`arxiv`、`pdf`

---

## 7. AI / LLM 集成

### 环境变量（`backend/.env`）

```env

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [li-xiu-qi/paper-graph-manager](https://github.com/li-xiu-qi/paper-graph-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
