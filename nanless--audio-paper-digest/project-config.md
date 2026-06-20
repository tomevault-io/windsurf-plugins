---
trigger: always_on
description: >
---


**[English](SKILL.en.md)** | 中文

# Paper Digest Skill（以当前代码为准）

## 1. 文档定位

- `SKILL.md`：给 Agent 的执行规则与安全约束
- `README.md`：给人的运行手册（命令、配置、排错）
- `prompts/filter.md`：筛选阶段 LLM prompt
- `prompts/deep-analysis.md`：深度分析阶段 LLM prompt（输出格式、标签体系、评分标准）

当文档与代码冲突时，**以 `scripts/*` 当前实现为准，并同步更新文档**。

---

## 2. 当前真实流程

主入口：`./run-full-fetch.sh`（或 `node scripts/full-fetch.js` / `npm run fetch`）

1. **自动归档**：检查 `data/current/deep-analysis-result.json` / `filtered-papers.json` / `analyzed.json`，若时间戳早于今天（北京时间）且 `data/archive/<日期>/` 下不存在，则复制后删除原文件。**`papers.json` 不归档。**
2. **加载去重库**：读取 `data/current/papers.json` 已有 ID；扫描 Hugo 博客仓库（`PAPER_DIGEST_BLOG_REPO`）中已发布论文的 arXiv ID，两者合并为统一去重集合
3. **arXiv 抓取**：7 个分类，每类最多 100 篇（可通过 `PD_ARXIV_MAX_RESULTS` 调整），遇连续 20 篇已有 ID 提前停止（去重集合包含 papers.json + 博客已发布 ID）
4. **HuggingFace 抓取**：`daily_papers` 分页（最多 20 页）+ `papers` API 补充，默认近 7 天，排除去重集合中的已有 ID
5. **合并去重**：arXiv 优先，HF 补充 7 个特有字段，标记 `sources`；过滤掉博客已发布论文
6. **LLM 筛选**：按 `PAPER_ANALYZER_*` 配置逐篇判断语音/音乐/音频相关，`batchSize=5`（可通过 `PD_FILTER_BATCH_SIZE` 调整），单篇超时 60 秒，重试 3 次
7. **保存筛选结果**：`data/current/filtered-papers.json`
8. **更新去重库**：追加所有爬取论文 ID 到 `data/current/papers.json`（不仅筛选通过的，提前保存防止后续中断丢失）
9. **深度分析**：`deep-analyzer.js`，全文+图片，并发 3 篇（可通过 `PD_ANALYSIS_CONCURRENCY` 调整），每篇最多重试 2 次（可通过 `PD_ANALYSIS_MAX_RETRIES` 调整）
10. **增量保存**：每批分析后立即保存到 `data/current/deep-analysis-result.json`，自带失败结果保护（已有成功 analysis 的论文不会被无 analysis 的失败结果覆盖）
11. **收尾合并**：去重合并历史结果，自动备份 bak 文件（保留最近 10 个）

`full-fetch.js` **不会自动发布博客/微信**，发布需单独运行 Python 脚本。

---

## 3. 数据路径规范

### 3.1 优先路径（当前）

| 文件 | 用途 | 归档行为 |
|------|------|---------|
| `data/current/papers.json` | 论文去重数据库 | **不归档**，持续累积 |
| `data/current/filtered-papers.json` | 筛选后的论文元数据 | 每日归档移走后重新生成 |
| `data/current/deep-analysis-result.json` | 核心分析结果（含 analysis / parsed / imageUrls） | 每日归档移走后重新生成 |
| `data/current/analyzed.json` | 旧版已分析记录（兼容） | 每日归档移走后重新生成 |

### 3.2 兼容行为

部分脚本在读取时兼容 `data/*.json` 旧路径，但新产物应写入 `data/current/`。

### 3.3 归档目录

`data/archive/<YYYY-MM-DD>/` 按日期子目录存放当日归档文件。`deep-analysis-result-<时间戳>.bak.json` 备份文件也存放在此目录下，自动清理保留最近 10 个。

---

## 4. 模型与环境变量

### 4.1 统一存放位置

**所有环境变量统一放在 `项目根目录的 `.env` 文件`。** `.zshrc` 已配置：
```zsh
set -a; source 项目根目录的 `.env` 文件 2>/dev/null; set +a
```

这意味着：
- shell 启动时自动注入所有变量
- Python 脚本直接通过 `os.environ` 读取
- Node 脚本通过 `loadEnvFile()` 二次兜底（仅补未设置的变量）

### 4.2 筛选阶段（`fetch-papers.js`）

筛选统一调用 `PAPER_ANALYZER_*` 指定的 LLM：

- endpoint: `PAPER_ANALYZER_ENDPOINT`（必填）
- key: `PAPER_ANALYZER_API_KEY`（必填）
- model: `PAPER_ANALYZER_MODEL`（必填）
- **API 协议自动路由**：`scripts/utils.js` 中的 `detectApiType()` 会根据端点和模型名自动判断使用 OpenAI 还是 Anthropic 协议
  - **MiMo/Kimi Token Plan / Coding Plan**（端点含 `token-plan` 或 `coding`，模型含 `mimo`/`kimi`）→ 自动切换为 **Anthropic 协议**，伪装成 Claude Code 调用
    - **MiMo**: `https://token-plan-cn.xiaomimimo.com/v1` → `https://token-plan-cn.xiaomimimo.com/anthropic/v1/messages`（替换 `/v1` 为 `/anthropic`）
    - **Kimi**: `https://api.kimi.com/coding/v1` → `https://api.kimi.com/coding/v1/messages`（直接加 `/messages`，无需 `/anthropic` 中间路径）
    - Headers: `x-api-key` + `anthropic-version: 2023-06-01` + `User-Agent: claude-cli/<version> (external, cli)`（版本号动态获取自本地 `claude --version`，失败回退到 `2.1.108`）
    - system message 自动提取为请求体顶级字段（Anthropic 要求）
  - **其他情况**（包括 MiMo 按量付费、通用 OpenAI 兼容端点）→ 使用标准 **OpenAI 协议**
    - URL: `/v1/chat/completions`
    - Headers: `Authorization: Bearer {key}`
- **agent: `false`** — LLM API 请求明确禁用连接复用，避免全局 agent 连接池被代理污染导致 MiMo 403（详见 9.2）
- 超时 60 秒，重试 3 次，每次重试独立创建 AbortController
- 指数退避：抓取 4s/8s/16s（`2^attempt * 2s`，上限 60s），限流 10s/20s/40s（`2^attempt * 5s`，上限 60s）
- prompt 来源：`prompts/filter.md`，运行时通过 `loadPrompt()` 读取并替换 `{title}`、`{abstract}`、`{categories}` 占位符
- 判定口径：多模态模型只要明确涉及语音/音乐/音频（输入、输出、训练目标、评测任务或核心能力之一）即判定为相关
- 冲突处理：若同时满足"多模态涉及语音/音乐/音频"和"其他领域"描述，优先判定为"是"

### 4.3 深度分析阶段（`deep-analyzer.js`）

深度分析统一使用 `PAPER_ANALYZER_*` 指定的 LLM，**与筛选阶段共用同一套 API 协议自动路由逻辑**：

- endpoint: `PAPER_ANALYZER_ENDPOINT`（必填）
- key: `PAPER_ANALYZER_API_KEY`（必填）
- model: `PAPER_ANALYZER_MODEL`（必填）
- `detectApiType()` 自动判断协议类型，行为与 4.2 节一致
  - **MiMo**: `/v1` → `/anthropic/v1/messages`
  - **Kimi**: `/coding/v1` → `/coding/v1/messages`

API 调用特性：
- 整体超时 20 分钟（AbortController）
- max_tokens=64000，temperature=0.7
- **双层重试**：analysis-engine.js 层面每篇最多重试 2 次（总共最多 3 次尝试）；deep-analyzer.js 内部每次 API 调用再重试最多 3 次（指数退避：第一次 10 秒，之后翻倍，`2^attempt * 5s`）
- **LLM API 请求明确设置 `agent: false`，强制直连以绕过本地代理（避免 MiMo 403）；arXiv/HuggingFace 等外部抓取仍使用代理自动检测**
- arXiv HTML 解析使用 **cheerio** 结构化选择器，移除 script/style/nav/header/footer 等噪音元素
- 图片下载 **并行化（并发 3）**，下载论文全部图片（无数量限制）；单张 base64 上限约 20M 字符（config.js 中 `imageMaxBase64Chars`）；超时后自动降级为纯文本重试
- 全文上限约 500K 字符（config.js 中 `fullTextMaxChars`）
- 所有分析配置集中管理于 `scripts/config.js`，支持环境变量覆写

输出约束：
- prompt 来源：`prompts/deep-analysis.md`，运行时通过 `loadPrompt()` 读取并替换 `{hasFullText}`、`{title}`、`{authors}`、`{categories}`、`{arxivId}`、`{textForAnalysis}` 占位符
- 固定一级标题：`## 评分`、`## 机器摘要`、`## 标签`、`## 作者与机构`、`## 毒舌点评`、`## 核心摘要`、`## 方法概述和架构`、`## 核心创新点`、`## 实验结果`、`## 细节详述`、`## 评分理由`、`## 局限与问题`、`## 开源详情`
- `## 评分` 下先输出总分（X.X/10）
- **代码后处理**：`parseAnalysis`/`parse_analysis` 会从 `## 评分理由` 中提取八个分项（创新性/2、技术严谨性/1.5、实验充分性/1.5、清晰度/1、影响力/1.5、开源/1.5、可复现性/0.5、工程/实践价值/1.5）重新计算总分，各分项之和上限为 10，四舍五入到 0.1，覆盖 LLM 原始总分

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanless/audio-paper-digest](https://github.com/nanless/audio-paper-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
