---
trigger: always_on
description: - 天枢智投是面向 A 股投研、AI 多智能体决策、模拟交易、长期记忆和经验复盘的研究系统。
---

# Best-AI-Trader 项目上下文

## 项目定位
- 天枢智投是面向 A 股投研、AI 多智能体决策、模拟交易、长期记忆和经验复盘的研究系统。
- 主栈：FastAPI + SQLAlchemy + PostgreSQL/Redis + LiteLLM Proxy + LangGraph/LangChain + React 18/Vite/TypeScript/Ant Design + MemoFlux。
- 完整部署由根 `docker-compose.yml` 编排 PostgreSQL、Redis、LiteLLM、Memory、Backend、Frontend、Nginx；源码调试使用 `docker-compose.dev.yml`。

## 关键目录
- `backend/app/main.py`：FastAPI 应用、lifespan 启停副作用、HTTP access log、CORS、WebSocket 挂载。
- `backend/app/api/__init__.py`：`/api/v1` 路由聚合与默认鉴权边界。
- `backend/app/ai/llm_engine/`：单股多 Agent 投研辩论工作流。
- `backend/app/ai/stock_picker/`：股票池、因子初排、整池 LLM 深研和推荐生成；不下单。
- `backend/app/ai/experience/`：已有 PM 决策的后验复盘、事件流和记忆写入。
- `backend/app/ai/agentic/`：Agent 工具、新闻插件、Skills Loader、Python 沙箱、浏览器、PDF、Memory 工具。
- `backend/app/data/`：A 股数据接入、DataFrame 落库、指标、刷新调度。
- `backend/app/models/`、`backend/app/crud/`、`backend/app/schemas/`：SQLAlchemy 模型、CRUD、Pydantic API 契约。
- `backend/app/trading/`：模拟交易服务与纯计算交易引擎。
- `backend/app/portfolio/`、`backend/app/risk_control/`、`backend/app/performance/`：组合估值、组合风控、绩效快照。
- `frontend/src/App.tsx`、`frontend/src/layouts/DashboardLayout.tsx`：前端路由与主布局。
- `frontend/src/api/`、`frontend/src/services/websocket.ts`、`frontend/src/utils/apiHistory.ts`：前端 API、WebSocket、任务历史。
- `memo/`：MemoFlux 独立子项目；主后端只通过 HTTP client 集成。
- `docs/`：正式编号设计/部署文档；`docs/superpowers/` 只作为临时工作流记录。

## 常用命令
- 后端默认测试：`pytest`（根 `pytest.ini` 默认只收集 `backend/tests`）。
- 后端定向测试：`pytest backend/tests/test_api_auth_required.py`。
- MemoFlux 测试：在 `memo/` 下运行 `pytest tests`。
- 前端质量门禁：`cd frontend && npm run lint && npm run typecheck && npm run build`。
- 本地镜像部署：`docker compose up -d`。
- 本地源码调试：`docker compose -f docker-compose.dev.yml up -d --build`。
- 配置变更后重建容器：`docker compose up -d --force-recreate <service>`，不要只用 `restart`。

## 修改前先读
- 部署/环境：`docs/01-deployment.md`，Windows WSL2 读 `docs/04-windows-wsl-docker-engine-deployment.md`。
- 后端能力地图：`backend/app/README.md`。
- 交易链路：`backend/app/trading/README.md`。
- Debate 工作流：`backend/app/ai/llm_engine/README.md`。
- AI 选股：`backend/app/ai/stock_picker/README.md`。
- 经验复盘：`backend/app/ai/experience/README.md`；注意该 README 中“最终 JSON 前必须调用 `write_memory`”的描述与当前代码/测试不一致，当前允许无新增可复用经验时不写 Memory，以 `backend/app/ai/experience/workflow.py` 和 `backend/tests/test_experience_workflow.py` 为准。
- Skills Loader：`backend/app/ai/agentic/skills_loader/README.md`。
- 新闻插件：`backend/app/ai/agentic/tooling/news_plugins/README.md`。
- 安全/合规/数据源：`SECURITY.md`、`LEGAL.md`、`DATA_SOURCES.md`、`CONTRIBUTING.md`。
- MemoFlux 变更：`memo/README.md`、`memo/docs/`。

## 常见任务路径
- 新增后端 API：先读 `backend/app/api/__init__.py`、相关 `schemas`/`models`/`crud`，默认加 `get_current_user`，补对应 `backend/tests`。
- 新增异步任务或调度：复用 `TaskManager`、`AsyncTaskRunner`、`AsyncTaskScheduler`，显式处理 `user_id`、状态和 Redis 通知。
- 新增前端页面：改 `frontend/src/App.tsx` 路由，页面放 `frontend/src/pages` 或既有 feature 下，API 放 `frontend/src/api/*.ts`。
- 改 LLM/Agent：先核对 LiteLLM 别名、Agent 工具边界、结构化输出 schema、usage 记录和相关 `test_llm_*`/agentic 测试。
- 新增新闻源：按 `backend/app/ai/agentic/tooling/news_plugins/README.md` 做插件，不新增平行工具。
- 新增数据源：按 `backend/app/data/ingestors/plugins/README.md` 做 ingestor，写入走 `DataIngestionService.write_dataframe()`。
- 改交易/组合/风控：先读 `backend/app/trading/README.md`，保持 API/Service/Engine 分层和风控预检。
- 改 Memory/经验复盘：先读 Memory 09/10 文档，语义判断走 LLM schema/prompt/eval，不写关键词规则。

## 架构约定
- 后端 LLM 接入固定走 LiteLLM Proxy 与模型别名，真实 provider key、模型名和 base URL 放在本地 `litellm/config.yaml`，不要写入后端代码或提交仓库；公开或多人环境还必须轮换 `general_settings.master_key` 并同步后端/Memory 使用的 LiteLLM API key。
- Debate 的事实上下文由 `backend/app/ai/llm_engine/context/` 构建；Agent 不应直接绑定数据库表结构拼 prompt。
- PM 是唯一追加交易工具的 Agent；普通分析师不应直接下单。
- `stock_picker` 只输出推荐、备选和风险摘要，不构建持仓组合、不执行交易。
- 后验评估统一落在 `experience` 复盘系统，不要另造平行历史评估中心。
- 手动/API 下单和 AI 下单都必须进入 `TradingService` 与 `TradingEngine`；`positions.purchase_details.ledger` 是 FIFO/T+1 关键账本。
- 组合估值复用 `build_portfolio_valuation`；组合风控复用 `portfolio_risk_control_service.evaluate_order`。
- 新闻源通过 `news_plugins` 插件体系接入，一个插件代表一个明确来源；不要新增平行新闻工具。
- Agent 使用 Skills Loader 时必须先 `load_skill` 再按需读 references/scripts；不能只看 catalog 猜用法。
- 主 backend 只能通过 `backend/app/ai/memory_client.py`/HTTP 集成 MemoFlux；不要直接写 MemoFlux 数据库。

## 复用入口
- 通用重试、日期、股票代码、JSON 安全序列化：`backend/app/core/utils/`。
- 结构化日志和敏感字段脱敏：`backend/app/core/logger.py`。
- 后端 i18n 文案加载与格式化：`backend/app/core/i18n.py`。
- 后端 `.env` 读写：`backend/app/core/env_manager.py`。
- 前端错误格式化：`frontend/src/utils/errorUtils.ts`。
- 前端 API 历史与脱敏：`frontend/src/utils/apiHistory.ts`。
- 前端 WebSocket 订阅：`frontend/src/services/websocket.ts`。

## 测试与夹具约定
- 后端测试默认使用 `backend/tests/conftest.py` 中的内存 SQLite、`client`、`db_session`、`auth_headers`。
- 新增后端 SQLAlchemy 表并参与 API/DB 测试时，必须检查并更新 `_sqlite_test_tables()`。
- 单测禁止访问真实 LLM、Redis、Tushare、NewsAPI、Tavily、浏览器外部服务；使用现有 mock/fake/monkeypatch 模式。
- Memory 测试环境设置 `MEMORY_DATABASE_URL=db.invalid`，真实数据库访问若未 mock 会失败。
- 提示词变更不要新增 pytest 或其他单元测试来约束 prompt 文案；提示词效果通过人工审计、既有评测脚本或用户明确要求的 live eval 验证。
- 前端当前没有 test script；提交前至少运行 lint、typecheck、build，且 ESLint warning 会因 `--max-warnings 0` 失败。

## 高风险边界
- 除 `/health`、`/api/v1/auth/register`、`/api/v1/auth/login`、`/api/v1/general/i18n/{lang}` 外，HTTP 业务路由默认必须要求登录；新增公开端点需同步测试白名单。
- WebSocket 禁止使用 JWT query token；必须先通过已鉴权 HTTP 换 30 秒一次性 ticket。
- `/api/v1/testing/*`、数据库备份/导入、数据源配置、新闻插件、Skills、运行时依赖安装都属于高风险面，必须保持鉴权。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarvekG/BestAITrader](https://github.com/MarvekG/BestAITrader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
