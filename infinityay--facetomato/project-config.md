---
trigger: always_on
description: |------|------|------|
---

# FaceTomato

## 变更记录 (Changelog)

| 时间 | 操作 | 说明 |
|------|------|------|
| 2026-03-28 | 增量扫描 | 同步首页语义调整：前端 `/` 改为面经题库首页，导航首项调整为题库，`/resume` 保留为简历解析入口 |
| 2026-03-23 | 增量扫描 | 前端本地持久化执行 cutoff：仅保留 `face-tomato-*` canonical key，移除旧品牌 key 自动迁移与 mock interview snapshot 版本兼容说明 |
| 2026-03-20 | 增量扫描 | 更新 backend 安装方式：RAG 依赖改为 `rag` 可选依赖，并补充 mock interview 在未安装 RAG 依赖时的自动回退说明、索引脚本前置条件与文档引用修正 |
| 2026-03-18 | 增量扫描 | 同步根目录 / frontend / backend 文档到当前实现：补充 runtime settings、语音转写、mock interview 本地恢复与 developer trace、RAG/non-RAG fallback、测试清单与路由面 |
| 2026-03-13 09:34:09 | 增量扫描 | 补充 mock interview、Vitest 测试、SSE/RAG/匿名会话恢复文档，覆盖率约 90.1% |
| 2026-03-04 16:22:25 | 初始化 | 首次生成项目文档，覆盖率约 85% |

---

## 项目愿景

FaceTomato（面柿）是一款面向求职场景的职业发展助手，围绕简历解析、JD 匹配分析、面经检索和模拟面试四条主链路，帮助用户从“准备材料”走到“实战演练”。

## 架构总览

- 前端：React 18 + TypeScript + Vite + TailwindCSS + Zustand + Vitest
- 后端：FastAPI + LangChain + 多 LLM 供应商支持 + SSE / WebSocket
- 数据存储：`sessionStorage` / `localStorage`、SQLite、ZVEC 本地检索索引
- 部署形态：前后端分离，Vite 开发服务器代理到 FastAPI

### 关键业务链路

1. 简历解析链路
   - 前端上传 PDF / DOC / DOCX / 图片 / 文本简历
   - 后端通过文档解析 + OCR（可选）+ LLM 抽取结构化 `ResumeData`
   - 支持请求级 runtime 配置覆盖默认模型/OCR 配置

2. JD 优化链路
   - 前端提交 JD 文本并先调用 `/api/jd/extract` 抽取结构化 `JDData`
   - 后端生成概览、建议与 JD-Resume 匹配结果（`/api/resume/jd/*`）
   - 支持请求级 runtime 配置覆盖

3. 面经题库链路
   - 后端基于 SQLite 提供分页、筛选、统计、公司聚合与邻近导航
   - mock interview 在创建阶段可引用题库检索证据

4. 模拟面试链路（当前实现）
   - 会话创建：`POST /api/mock-interview/session/stream-create`（SSE）
   - 对话续流：`POST /api/mock-interview/session/{sessionId}/stream`（SSE）
   - 前端通过 `localStorage` 当前快照结构恢复，后端按请求体重建临时会话态（ephemeral）
   - 支持 developer context / developer trace 事件（retrieval、plan_generation、reflection、interviewer_generation）
   - 当 `MOCK_INTERVIEW_RAG=false`、未安装 `rag` 可选依赖，或 RAG 运行时依赖不可用时，自动退化到 non-RAG 检索策略

5. 语音输入链路
   - 前端先调用 `/api/speech/status` 检查可用性
   - 前端通过 `WS /api/speech/transcribe` 上传 PCM 音频并接收 partial/final 识别结果
   - 支持 runtime speech key 覆盖默认后端配置

## 模块结构图

```mermaid
graph TD
    A["(根) FaceTomato"] --> B["frontend"]
    A --> C["backend"]

    B --> B1["src/pages"]
    B --> B2["src/store"]
    B --> B3["src/lib"]
    B --> B4["src/components"]

    C --> C1["app/api/routes"]
    C --> C2["app/services"]
    C --> C3["app/schemas"]
    C --> C4["app/prompts"]
    C --> C5["app/core"]

    click B "./frontend/CLAUDE.md" "查看 frontend 模块文档"
    click C "./backend/CLAUDE.md" "查看 backend 模块文档"
```

## 模块索引

> 当前前端首页语义：`/` 进入面经题库，`/resume` 为简历解析独立入口。

| 模块 | 路径 | 语言 | 职责 |
|------|------|------|------|
| frontend | `frontend/` | TypeScript / React | 用户界面层，负责简历上传预览、JD 优化交互、面经题库浏览、模拟面试会话（本地恢复/导出）与语音输入 |
| backend | `backend/` | Python / FastAPI | API 与服务层，负责 OCR/LLM 抽取、JD 匹配、题库查询、RAG 检索、mock interview SSE 与语音转写 |

## 运行与开发

### 前端

```bash
cd frontend
npm install
npm run dev
npm run build
npm run test:run
```

- 开发端口：`5569`
- 代理目标：`http://127.0.0.1:6522`

### 后端

```bash
cd backend
uv sync
cp .env.example .env
uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 6522
```

默认安装不会拉取本地 RAG 大依赖；如果你要启用 mock interview RAG 或执行索引脚本，请先执行：

```bash
cd backend
uv sync --extra rag
```

可选脚本：

```bash
cd backend
uv run python scripts/migrate_db.py --source-dir <path-to-interview-json-root>
uv run python scripts/build_interview_zvec_index.py --help
```

### 环境变量

后端主要配置位于 `backend/.env`，可从 `backend/.env.example` 复制。重点变量：

- 应用与上传
  - `APP_HOST`、`APP_PORT`、`CORS_ORIGINS`、`MAX_UPLOAD_MB`

- LLM 供应商与模型
  - `MODEL_PROVIDER`
  - `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENAI_MODEL`
  - `GOOGLE_API_KEY`、`GOOGLE_MODEL`
  - `ANTHROPIC_API_KEY`、`ANTHROPIC_MODEL`

- OCR
  - `ZHIPU_APIKEY`

- 语音
  - `VOLCENGINE_SPEECH_BASE_URL`、`VOLCENGINE_SPEECH_MODE`
  - `VOLCENGINE_SPEECH_APP_KEY`、`VOLCENGINE_SPEECH_ACCESS_KEY`
  - `VOLCENGINE_SPEECH_RESOURCE_ID`

- 模拟面试
  - `MOCK_INTERVIEW_RAG`
  - `MOCK_INTERVIEW_SESSION_TTL_MINUTES`
  - `MOCK_INTERVIEW_PLAN_TIMEOUT_SECONDS`

- 题库与检索
  - `INTERVIEW_DB_PATH`、`INTERVIEW_ZVEC_INDEX_PATH`
  - `INTERVIEW_RAG_TOPK`、`INTERVIEW_RAG_CANDIDATE_TOPK`
  - `INTERVIEW_RAG_DENSE_WEIGHT`、`INTERVIEW_RAG_SPARSE_WEIGHT`
  - `INTERVIEW_DENSE_EMBEDDING_*`、`INTERVIEW_SPARSE_EMBEDDING_*`

> 说明：前端 runtime settings（LLM/OCR/Speech）通过请求参数传入后端，再由 `runtime_config` 服务与上述 env 默认值逐字段合并。

## 测试策略

### 前端（Vitest + Testing Library）

- 页面测试
  - `frontend/src/pages/__tests__/DiagnosisPage.test.tsx`
  - `frontend/src/pages/__tests__/MockInterviewPage.test.tsx`

- 应用壳层/运行时设置
  - `frontend/src/test/App.test.tsx`

- Resume 组件
  - `frontend/src/components/resume/__tests__/ResumeParsingState.test.tsx`
  - `frontend/src/components/resume/__tests__/ResumeExtractPanel.test.tsx`

- Optimization 组件
  - `frontend/src/components/optimization/__tests__/AnalysisPhase.test.tsx`
  - `frontend/src/components/optimization/__tests__/ResumeDisplayPanel.test.tsx`
  - `frontend/src/components/optimization/__tests__/SuggestionCard.test.tsx`

- Store / Lib
  - `frontend/src/store/__tests__/resumeStore.test.ts`
  - `frontend/src/store/__tests__/runtimeSettingsStore.test.ts`
  - `frontend/src/lib/__tests__/api.test.ts`

### 后端（pytest）

- 简历解析与路由
  - `backend/tests/test_pdf_parser.py`
  - `backend/tests/test_resume_routes.py`

- 优化与匹配
  - `backend/tests/test_resume_optimization_routes.py`
  - `backend/tests/test_resume_optimizer.py`
  - `backend/tests/test_jd_optimization_routes.py`
  - `backend/tests/test_jd_resume_matcher.py`

- 题库与检索

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infinityay/FaceTomato](https://github.com/Infinityay/FaceTomato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
