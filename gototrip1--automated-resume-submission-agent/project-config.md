---
trigger: always_on
description: > 本文档是项目的「指挥中心」，所有协作者（包括 Claude）在动手前都应先阅读本文。
---

# 自动投递简历 Agent 项目 (interviewAccessWebsite)

> 本文档是项目的「指挥中心」，所有协作者（包括 Claude）在动手前都应先阅读本文。
> 任何架构、模块、字段的变更都必须同步更新本文档。

---

## 1. 项目概述 (Project Overview)

构建一个智能投递简历的 Agent 系统，用户上传简历并配置筛选条件后，系统自动从国内主流招聘 / 校招网站（Boss 直聘、实习僧、牛客、应届生求职网、智联招聘、前程无忧、拉勾、猎聘等）抓取符合条件的岗位，并代为投递简历，最终向用户回传投递结果（成功列表、失败列表与失败原因）。

### 核心价值
- 把求职者从「在 N 个 App 里反复搜索 + 点投递」中解放出来。
- 用 LLM + LangGraph 编排决策流程，让岗位匹配比关键词搜索更贴近用户真实意图。
- 给出可审计的投递清单和失败原因，便于复盘。

### 合规与边界 (重要)
- 各招聘平台普遍禁止自动化投递，本项目仅作为**技术研究 / 学习用途**使用。
- 默认不破解登录态、不绕过验证码、不规避平台风控；登录态由用户手动完成 (扫码 / 短信验证码)，Agent 仅在用户已登录的浏览器上下文内执行操作。
- 所有抓取行为应遵循目标站点的 robots.txt 与服务条款；生产部署需用户自担风险。
- 简历是敏感个人数据，必须做到：本地或加密存储、不外发到第三方、可一键删除。

---

## 2. 技术栈 (Tech Stack)

### 后端
- **语言**: Python 3.11+
- **Agent 编排**: LangGraph (核心工作流)
- **LLM 调用**: LangChain (OpenAI / Anthropic / 通义千问，可配置)
- **Web 框架**: FastAPI (对外 REST + WebSocket)
- **异步任务**: asyncio + (可选 Celery / RQ，根据规模决定)
- **浏览器自动化**: Playwright (推荐，支持反检测 + 多上下文)
- **HTML 解析**: BeautifulSoup4 / lxml / parsel
- **简历解析**: pdfplumber + python-docx + LLM 兜底解析
- **数据校验**: Pydantic v2
- **数据库**: SQLite (MVP) → PostgreSQL (生产)
- **ORM**: SQLAlchemy 2.x + Alembic
- **认证**: JWT (python-jose) + passlib bcrypt
- **日志**: loguru
- **测试**: pytest + pytest-asyncio

### 前端
- **框架**: Vue 3 (Composition API + `<script setup>`)
- **UI 库**: Element Plus
- **构建**: Vite
- **语言**: TypeScript
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **HTTP**: Axios (含拦截器：JWT 注入、401 跳登录)
- **实时反馈**: WebSocket 或 SSE，用于推送投递进度

---

## 3. 系统架构 (Architecture)

```
┌────────────┐    HTTP/WS    ┌──────────────────────────────┐
│  Vue 前端   │ ─────────────▶│  FastAPI 网关层              │
│ ElementPlus│ ◀─────────────│  (认证 / 路由 / WS 推送)     │
└────────────┘                └──────────────┬───────────────┘
                                             │
                                             ▼
                              ┌──────────────────────────────┐
                              │   LangGraph Agent 编排层     │
                              │  ┌────────────────────────┐  │
                              │  │ Resume Parser Node     │  │
                              │  │ Intent Extract Node    │  │
                              │  │ Site Selector Node     │  │
                              │  │ Search & Crawl Node    │  │
                              │  │ Fuzzy Match Node (LLM) │  │
                              │  │ Filter Node            │  │
                              │  │ Apply Node             │  │
                              │  │ Aggregator Node        │  │
                              │  └────────────────────────┘  │
                              └──────────────┬───────────────┘
                                             │
                          ┌──────────────────┼──────────────────┐
                          ▼                  ▼                  ▼
                   ┌────────────┐    ┌────────────┐      ┌────────────┐
                   │ Site Adapter│   │ Site Adapter│      │ Site Adapter│
                   │  Boss 直聘  │    │   实习僧    │  ...│   牛客      │
                   └─────┬──────┘    └─────┬──────┘      └─────┬──────┘
                         └──────────────────┴──────────────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │  Playwright     │
                                   │  浏览器池        │
                                   └─────────────────┘

                              ┌──────────────────────────────┐
                              │     存储层 (SQLite/PG)        │
                              │  用户 / 简历 / 任务 / 投递记录│
                              └──────────────────────────────┘
```

---

## 4. LangGraph Agent 工作流 (核心)

State (TypedDict) 至少包含：
```python
class AgentState(TypedDict):
    user_id: str
    raw_resume: bytes | str         # 原始简历内容
    parsed_resume: dict             # 结构化简历
    user_filters: dict              # 用户筛选条件
    target_sites: list[str]         # 选定的目标站点
    candidate_jobs: list[Job]       # 抓取到的候选岗位
    matched_jobs: list[Job]         # 过滤后匹配的岗位
    apply_results: list[ApplyResult]# 投递结果
    errors: list[ErrorRecord]
    progress: ProgressEvent         # 用于 WS 推送
```

### 节点 (Nodes)
1. **resume_parser**: PDF/DOCX → 结构化 JSON (姓名、学历、技能、经历、期望岗位)。
2. **intent_extractor**: 结合用户输入的「岗位名称」+ 简历，用 LLM 提炼搜索关键词与岗位画像。
3. **site_selector**: 根据用户选择 / 默认策略决定要抓哪些站点。
4. **search_and_crawl** (并行 fan-out): 调用各站点 Adapter 的 `search()`，返回岗位列表。
5. **fuzzy_matcher**: 用 LLM + 规则给每个候选岗位打分 (RapidFuzz + 嵌入相似度兜底)，过滤明显不符。
6. **filter_node**: 按硬性条件 (学历、薪资、规模、融资、地点、到岗天数、岗位类型) 过滤。
7. **apply_node** (并行 + 速率限制): 调用各 Adapter 的 `apply(job, resume)`，记录成功/失败原因。
8. **aggregator**: 汇总结果，写 DB，推 WS。

### 控制流
- 节点之间用 `add_conditional_edges` 处理：搜不到岗位 → 终止 + 提示；匹配数为 0 → 反问用户是否放宽条件。
- 失败重试：节点级 try/except + 最多 N 次退避重试；不可恢复错误立即写 `errors` 并继续。

---

## 5. 模块拆解 (Module Breakdown)

### 后端模块
| 模块 | 路径 | 职责 |
|------|------|------|
| API 层 | `backend/app/api/` | FastAPI 路由 (auth / resume / task / result / ws) |
| 认证 | `backend/app/auth/` | JWT 签发、密码哈希、依赖注入 `get_current_user` |
| 简历解析 | `backend/app/services/resume_parser.py` | PDF / DOCX → 结构化字段；LLM 兜底 |
| 任务编排 | `backend/app/agent/graph.py` | LangGraph 图定义 + 状态 |
| Agent 节点 | `backend/app/agent/nodes/` | 每个节点一个文件，独立可测 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gototrip1/Automated-resume-submission-Agent](https://github.com/gototrip1/Automated-resume-submission-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
