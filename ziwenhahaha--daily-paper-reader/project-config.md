---
trigger: always_on
description: > 本文件为 AI 助手（Claude / Codex 等）提供项目上下文，帮助快速理解代码库。
---

# CLAUDE.md — Daily Paper Reader 项目导读

> 本文件为 AI 助手（Claude / Codex 等）提供项目上下文，帮助快速理解代码库。

## 一、项目定位

**Daily Paper Reader** 是一个 fork-and-run 的个人/实验室每日论文推荐+阅读器。

- **零服务器**：GitHub Actions 定时跑后端流水线，GitHub Pages 部署前端站点
- **Fork 即用**：Fork 后配好 DeepSeek API Key + GitHub PAT，5 分钟上线
- **全链路自动化**：arXiv/会议论文抓取 → 个性化召回 → 重排 → LLM 精炼打分 → 生成可阅读日报
- **纯前端架构**：订阅管理、密钥存储、AI 问答、Zotero 集成全在浏览器端完成

## 二、整体架构分层

```
┌────────────────────────────────────────────────────────────┐
│  前端层 (app/)     docsify 论文站点 + 订阅管理 + AI 问答    │
│                   + Zotero 集成 + workflow 触发面板          │
├────────────────────────────────────────────────────────────┤
│  编排层            GitHub Actions (8 个 workflow)           │
│  (.github/)        + 本地调试后端 (local_debug_server.py)    │
├────────────────────────────────────────────────────────────┤
│  流水线层          main.py 编排 Step 0→6                     │
│  (src/)            配置→抓取→召回→排序→精炼→选择→生成文档      │
├────────────────────────────────────────────────────────────┤
│  数据维护层         maintain/ 多源抓取+向量编码+Supabase 同步 │
│  (src/maintain/)   arxiv/bioRxiv/medRxiv/ChemRxiv/会议      │
├────────────────────────────────────────────────────────────┤
│  存储层            Supabase (PostgREST + pgvector + FTS)    │
│  (sql/)            11 张同构论文表 + 向量/BM25 RPC           │
└────────────────────────────────────────────────────────────┘
```

## 三、端到端数据流（每日运行一次）

```
定时触发 (UTC 18:30 / 北京 02:30)
  │
  ├─ [Step 0] 0.enrich_config_queries.py   (可选) LLM 扩充关键词
  │
  ├─ [Step 1] fetch_arxiv.py              (可跳过) 从 arXiv 抓新论文
  │       ↓ 若 Supabase 已接管 → 直接跳过
  │
  ├─ [Step 2.1] BM25 关键词召回            Supabase RPC match_*_bm25
  ├─ [Step 2.2] 向量语义召回               Supabase RPC match_*_exact
  ├─ [Step 2.3] RRF 融合                  两路排名融合 → 统一候选池
  │       ↓ archive/<date>/filtered/
  │
  ├─ [Step 3] Reranker 重排               Qwen3-Reranker 本地/远端
  │       ↓ archive/<date>/rank/
  │
  ├─ [Step 4] LLM 精炼打分                DeepSeek 0-10 相关性评分
  │       ↓ archive/<date>/rank/*.llm.json
  │
  ├─ [Step 5] 选择论文                    精读区(≥8分) + 速读区(6-8分)
  │       ↓ archive/<date>/recommend/
  │
  └─ [Step 6] 生成文档                    docsify markdown + 日报 + 图表
          ↓ docs/<date>/ + docs/_sidebar.md + docs/README.md
          ↓ git commit + push → GitHub Pages 自动部署
```

**关键设计**：各 Step 间通过 `DPR_RUN_DATE` 环境变量隐式共享 archive 路径（`archive/<date_token>/{raw,filtered,rank,recommend,logs}/`），`main.py` 只设一次这个 env，各脚本顶部独立重算路径。

## 四、各子系统详解

### 1. 流水线编排 `src/main.py`

- **纯顺序编排器**：`run_step()` = `subprocess.run(check=True)`，任一步失败整条中断
- **三种模式**：
  - **standard**（默认，≤9 天窗口）：正常精读推荐
  - **skims**（≥11 天窗口）：速览模式，全部走 quick_skim，忽略 seen_ids
  - **long-range**（≥10 天）：生成区间 token `YYYYMMDD-YYYYMMDD`
- **跳过抓取**：当 Supabase 完全接管（BM25 + 向量 RPC 均启用）时自动跳过 Step 1
- **关键函数**：`should_skip_fetch()`、`resolve_run_date_token()`、`use_skims_mode()`、`resolve_summary_step_env()`

### 2. 订阅与查询规划 `src/subscription_plan.py`

- **唯一真源**：把 `config.yaml` 的 `subscriptions.intent_profiles`（每个研究方向含 keywords + intent_queries）翻译成检索计划
- **keyword 走 BM25 召回，intent_query 走向量召回**——召回词 vs 语义句的刻意拆分
- **paper_tag 命名空间**：`keyword:<tag>` / `query:<tag>`，命中即给论文打 tag
- **输出**：`build_pipeline_inputs()` → `{bm25_queries, embedding_queries, context_queries}`

### 3. 三路召回与融合 `src/2.1` + `2.2` + `2.3`

| 步骤 | 方法 | 实现 |
|------|------|------|
| 2.1 | BM25 关键词 | Supabase FTS RPC `match_*_bm25`，0 命中回退本地 |
| 2.2 | 向量语义 | Supabase 精确余弦 RPC `match_*_exact`，bge-small-en-v1.5 |
| 2.3 | RRF 融合 | `1/(60+rank)` 跨路合分，Top-200 截断 |

- **自适应 top_k**：窗口每满 1000 篇 +50（基数 50）
- **超时兜底**：PG `statement_timeout` 57014 → 时间窗递归二分重试
- **多源路由**：`source_backend_router.group_queries_by_source` 按 `paper_sources` 扇出到不同 Supabase 表

### 4. 排序/Rerank `src/3.rank_papers.py`

- **双层 RRF**：lane 间合池 + query 内跨批融合，均用 `1/(60+rank)`
- **Reranker 三选一**：
  - 本地 `Qwen3-Reranker-0.6B`（yes/no token logits → P(yes) 为分数）
  - 远端 SiliconFlow API（`src/reranker_api.py`）
  - 远端 zwwen.online 公益服务
- **星级是相对量**：min-max 归一化，每个 query 必有一个 5★ 和 1★
- **动态预算**：`lane_top_k = min(30 + 10 * ((total-1) // 1000), 120)`
- **离线评估**：`rerank_budget_experiment.py`（预算 profile 对比）、`rerank_model_size_experiment.py`（模型尺寸对比）

### 5. LLM 精炼 + 选择 `src/4.llm_refine_papers.py` + `src/5.select_papers.py`

- **Step 4**：只处理 ≥4★ 候选（约 RRF ≥0.5），DeepSeek 给 0-10 精排分 + 中英文摘要
- **Step 5**：分层选片——≥8 进精读区，6-8 进速读区，≥9 优先且突破 cap
- **名额** = base + tag_count（订阅 tag 越多名额越大）
- **carryover 机制**：`archive/carryover.json` 跨天结转高分论文

### 6. LLM 客户端封装 `src/llm.py`

- **OpenAI 兼容**：`LLMClient` / `DeepSeekClient`
- **结构化输出三级回退**：`json_schema → json_object → prompt_only`
- **JSON 截断自动修复**：`_repair_json_suffix` 处理 max_tokens 截断
- **多 base_url 重试**：`_iter_retry_bases` 遍历候选端点
- **注意**：每篇论文独立 client 实例，避免多线程下 `client.kwargs` 竞争

### 7. 文档生成 `src/6.generate_docs.py` + `src/paper_figures.py`

- **每篇一个 markdown**：YAML front matter（前端契约）+ 速览五段 + 精读长总结 + PDF 图表
- **并发生成**：`ThreadPoolExecutor`，每篇独立 `DeepSeekClient` 避免多线程竞争
- **图表提取**：PaperCropper + DocLayout-YOLO → PyMuPDF 兜底，全链 `continue-on-error`
- **幂等可重入**：`upsert_auto_block` / `upsert_front_matter_field` 只改目标段不重写全文
- **产物路径**：`docs/<date>/<paper_id>.md`、`docs/<date>/figures/`、`docs/README.md`、`docs/_sidebar.md`

### 8. 多源后端抽象 `src/source_config.py` + `source_backend_router.py` + `supabase_source.py`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziwenhahaha/daily-paper-reader](https://github.com/ziwenhahaha/daily-paper-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
