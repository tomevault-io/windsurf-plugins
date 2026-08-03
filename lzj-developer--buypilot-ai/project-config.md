---
trigger: always_on
description: **BuyPilot-AI**：基于 RAG 的多模态电商智能导购 Agent
---

# BuyPilot-AI 项目开发指引
## 项目概要

**BuyPilot-AI**：基于 RAG 的多模态电商智能导购 Agent
**赛事**：ByteDance AI Full-Stack Challenge（3 周 / 3 人）
**一句话定位**：把用户模糊购物需求转化为可解释决策路径的多品类智能导购决策智能体

| 决策项 | 选择 |
|--------|------|
| 品类 | 多品类（美妆护肤/数码电子/服饰运动/食品生活），导师提供官方数据 |
| 客户端 | Android 原生（Kotlin + Jetpack Compose + OkHttp SSE 直连） |
| LLM | **百炼主力 + Doubao 兜底**：qwen-turbo 做意图/标准/推荐/决策主力，Doubao 做 fallback；Qwen-VL-Plus 做图片理解 |
| 后端 | Python FastAPI + PostgreSQL + pgvector + SQLModel |
| 流式协议 | SSE（OkHttp SSE 直连 FastAPI `/chat/stream`） |
| 模型切换 | task-oriented interface（analyze_intent / generate_criteria / generate_recommendation / analyze_image），内部按 TASK_MODEL_MAP 选 primary/fallback |

### 模型-任务映射（百炼主力策略）

| 任务 | Primary | Fallback | 选型原因 |
|------|---------|----------|---------|
| 意图识别/路由 | Qwen-Turbo | Doubao-Seed-2.0-lite | qwen-turbo 延迟低、成本低，实测效果与 Doubao 无显著差异 |
| 购买标准生成 | Qwen-Turbo | Doubao-Seed-1.6 | 结构化 prompt + JSON schema 约束下，qwen-turbo 输出稳定 |
| 推荐解释生成 | Qwen-Turbo | Doubao-Seed-1.6 | 推荐文案质量在 prompt 工程下与 qwen-plus 差异不大，成本优势明显 |
| 最终决策生成 | Qwen-Turbo | Doubao-Seed-1.6 | 决策基于候选 + 反馈的结构化输入，模型能力要求不高 |
| 多模态图片理解 | Qwen-VL-Plus | 无 | Doubao VL 生态不确定，Qwen-VL-Plus 成熟 |
| Embedding | text-embedding-v3 (1024维) | 确定性 fallback (16维) | 百炼有 Key + 维度确定；fallback 仅供开发态 |
| Rerank | qwen3-rerank (百炼) | 无 | Doubao 无对应服务 |

> Doubao API 配置见 `.env.example`（含 BaseAPI / Model / Key / Limit）。模型可自由选择，上表是当前决策。

## 项目核心约束

### P0/P1/P2 — 目标优先级

**P0 = 基础功能完整性(35%) + 工程质量底线(25%)**
- 用户拿起App能走通完整链路：输入文字 → 意图识别 → 购买标准生成(初版) → 检索 → 推荐生成(初版) → SSE流式返回 → 商品卡片渲染
- 代码结构清晰、接口设计合理、错误处理完善、文档齐全
- docker-compose up + Android Studio Run 一键启动
- 依赖版本锁定(requirements.txt固定版本) + 核心逻辑注释(RAG链路/Prompt构造)

**P1 = 效果与可靠性(20%) + 加分项启动(20%)**
- 流畅、美观、无Bug；检索准确率、无幻觉、复杂场景处理
- 做精一项胜过浅尝三项
- 正式版卡片渲染(标准卡/商品卡/决策卡) + 流式动画
- 混合检索(硬过滤+向量+Rerank) + 证据绑定
- 多轮上下文 + 反馈记录 + 反选排除
- 图片上传 + Qwen-VL-Plus理解
- 对话式加购入门(cart_action SSE事件 + cart_items表)
- 双轨模型+fallback机制

**P2 = 打磨 + 稳定性**
- Thinking骨架屏/流式动画打磨
- 卡片交互细节打磨(滑动/收藏状态变化)
- 评测页面最小版(4个核心指标数字+版本对比)
- 4条Demo路径稳定性打磨

### 绝对不做（明确裁剪）

- 真实支付流程（挑战档不做，只做轻量确认购买意向闭环）
- 地址管理与物流状态
- 真实订单中心与退款售后
- 商家入驻与商品管理后台
- Multi-Agent 蜂群架构
- 长期用户画像系统
- 全量真实电商平台接入
- Celery + Redis 异步任务
- JWT 复杂认证
- 完整商家后台（订单/库存/支付/权限）
- 语音输入（多模态入门档，已有拍照找货覆盖）
- 知识图谱增强检索（GraphRAG）
- 热门查询缓存/首屏极速优化（工程质量加分方向不做）
- 完整Web管理后台（只做最小评测页面）

### 加分项策略

| 加分方向 | 目标档位 | 定位 |
|----------|---------|------|
| 4.2 多模态交互 | 拍照找货 | 深度方向1 |
| 4.3 对话智能与RAG增强 | 反选排除 + 多商品对比 | 深度方向2 |
| 4.1 业务闭环 | 入门对话式加购 | 轻量覆盖方向 |
| 4.4 工程质量与性能 | **不做** | 不投入 |

### 数据策略

- **100条**商品数据（导师提供的脱敏电商数据，4品类×25，中文）
- 每条含：product_id, title, brand, category, sub_category, base_price, image_path, skus(多规格), rag_knowledge{marketing_description, official_faq[], user_reviews[]}
- rag_knowledge 天然 chunking：marketing_description + 每个 FAQ + 每个 review = 独立 chunk
- 数据质量比数据数量重要
- products 表 Schema 使用 metadata JSONB 承载品类结构化属性（如护肤品的肤质适用、数码的存储规格等）

### 止损规则（里程碑驱动）

| 时间节点 | 验收标准 | 否则 |
|----------|---------|------|
| 5/27（第 7 天） | **完整链路能走通** + 一键启动 | 砍掉所有P1/P2，只追P0到能跑为止 |
| 6/03（第 14 天） | **4条Demo路径全部可演示** + 无幻觉无Bug | 砍掉复杂后台和增强功能 |
| 6/07（第 18 天） | 冻结功能 + design-decisions.md完成 | 只修Bug、打磨体验、准备答辩 |

### 减分项防御

| 减分项 | 防御措施 |
|--------|---------|
| AI编造商品/价格/优惠（幻觉） | 混合检索+硬过滤+证据绑定 |
| 纯Web/H5替代原生App | Android原生开发 |
| Demo无法运行或需大量手动配置 | docker-compose up一键启动 + README运行说明 |
| 代码完全依赖AI生成无法解释原理 | design-decisions.md + 答辩前自读一遍，能脱口而出每个核心决策的why |

### 四条 Demo 路径

| # | Demo路径 | 品类 | 核心演示能力 | 对应官方场景 |
|---|---------|------|-------------|-------------|
| 1 | "推荐适合油皮的洗面奶，200元以内" | 美妆护肤 | 模糊推荐+条件筛选 | 单轮模糊推荐 + 条件筛选 |
| 2 | 上传护肤品图片+"这个适合敏感肌吗？" | 美妆护肤 | 拍照找货+VL理解 | 拍照找货 |
| 3 | "不要含酒精的防晒霜"+"预算降到200" | 美妆护肤 | 反选排除+多轮约束 | 多轮追问 + 反选排除 |
| 4 | "把这个加到购物车" | 通用 | 对话式CRUD+加购入门 | 购物车入门 |

---
## 文档索引

所有文档在 `doc/` 目录下，按语义分层：

```
doc/
├── strategy/          战略层（为什么做）
│   ├── 01-比赛背景与战略决策.md    ← 比赛背景 + 最终决策
│   └── 02-策略研究报告.md         ← 完整战略报告，MVP 边界，Demo 路径
├── prd/               产品需求（做什么）
│   ├── 01-Android前端PRD.md       ← Android 客户端执行文档（SSE 事件、卡片规范、Kotlin 契约）
│   └── 02-后端与AgentPRD.md       ← 后端 & Agent 执行文档（数据库 Schema、管道编排、API 契约）
├── status/            完成状态（做到哪了）
│   └── backend-completion.md      ← 后端功能完成状态，按 P0/P1/P2 分层，AI 每次开发后自动更新
├── research/          调研参考
│   └── 队友原始调研-多模态电商导购.md
├── risk/              风险预判
│   └── 卡点与风险清单.md          ← 10+ 卡点 + 止损规则
└── prompts/           提示词工具包
    ├── 01-DeepResearch提示词.md    ← 索引版
    ├── 02-DeepResearch提示词拆分.md ← 12 个分段 prompt
    └── 03-AI编码助手系统提示词.md  ← Linus 角色 + AGENTS.md 架构 + 编码原则
```

**编码相关指引**：
- 前端接口契约 → `doc/prd/01-Android前端PRD.md`（SSE 事件类型、Kotlin data class、卡片规范）
- 前端表格附录 → `doc/prd/01-附录-表格内容.md`（错误码、状态机、渲染策略、测试用例等）
- 后端接口契约 → `doc/prd/02-后端与AgentPRD.md`（数据库 Schema、管道编排、API 端点）
- 后端 Prompt 模板 → `backend/prompts/`（已由 `PromptStore` 运行时加载；优先改 .md 文件，硬编码字符串在 `llm_task_payloads.py` 中仅作 fallback）
- 产品定义 → `PRODUCT.md`（用户画像、产品目的、设计原则、Anti-references）
- 设计系统 → `DESIGN.md`（颜色、字体、间距、圆角、组件规范，开发 UI 时必须遵循）
- 风险清单 → `doc/risk/卡点与风险清单.md`（开发前必读）
- 设计决策 → `design-decisions.md`（每个核心决策 3 句话：选了什么/为什么/反过来会怎样）
- 后端完成状态 → `doc/status/backend-completion.md`（P0/P1/P2 功能完成度，AI 每次开发后更新）
- 前后端契约 → `contracts/sse-events.schema.json`（SSE 事件 JSON Schema，铁律 1 的 source of truth）
- 实现差距与踩坑 → `260524-handoff.md`（实现 vs PRD 差距表 + 已知陷阱）
- 评测模块 → `eval-module-handoff.md`（评测设计决策 + 运行方式）

## Codemap
### 分层架构
```
UI → Runtime → Service → Repo → Config/Types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LZJ-Developer/BuyPilot-AI](https://github.com/LZJ-Developer/BuyPilot-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
