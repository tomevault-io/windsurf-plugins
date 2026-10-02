---
trigger: always_on
description: 本文件是本仓库**最高优先级的项目约定**。所有 AI 编码代理（Codex/Claude/其他）与人类协作者都必须遵守。
---

# AGENTS.md

本文件是本仓库**最高优先级的项目约定**。所有 AI 编码代理（Codex/Claude/其他）与人类协作者都必须遵守。

- 若与其他文档、注释、口头约定冲突，**以本文件为准**。
- 若任务需要改变本文件中标注为「架构级决策」的内容，**必须在同一次改动中先更新本文件**，再改代码。
- 本文件描述「项目要建成什么 + 怎么建」，不是当前进度快照。进度与待办写在 `docs/SPEC.md` 与 issue 中。

---

## 1. 项目目标

构建一个**基于 DeepSeek V4 的 literature briefing 多 Agent 系统**：

> 输入一个研究 topic，检索 3–5 篇相关论文，逐篇分析，最终生成一份可引用的 PDF 报告。

主入口（目标形态，实现时不得随意改名）：

```bash
briefing run --topic "diffusion models for weather forecasting" --out outputs/2026-09-17-weather
```

### 1.1 产品级完成标准（Definition of Done）

一次运行结束后，`<out>/` 目录必须同时存在：

| 文件 | 内容 |
| --- | --- |
| `report.pdf` | 最终交付物，见 §8 报告规范 |
| `briefing.md` | 报告的可编辑版本。它与 PDF **同源于同一个通过校验的 `Briefing` 对象**（而非 PDF 的输入），因此"两者内容一致"由构造保证 |
| `briefing.json` | 结构化简报（章节、结论、对比表） |
| `papers.json` | 入选论文的完整元数据 + 逐篇分析结果 |
| `manifest.json` | 运行审计：模型 id、prompt 版本、参数、token 用量、耗时、检索来源、缓存命中情况、renderer |

硬性验收条件：

1. 论文数量 **3 ≤ n ≤ 5**；不足 3 篇时**明确失败并说明原因**，绝不允许凑数或编造。
2. 每篇论文都具备可解析的：标题、作者列表、年份、来源（venue / 预印本平台）、URL，以及 DOI 或 arXiv ID（至少其一，确实不存在时显式标注 `null` 并说明）。
3. `report.pdf` 中**每一条引用都能追溯到 `papers.json`**；零虚构引用（详见 §3 Verifier）。
4. PDF 中英文混排可正常阅读，字体内嵌，文本可选中、可搜索，页码与目录正确。
5. 单次运行有可审计的调用次数与 token 上限（§11）。
6. `manifest.json` 能回答「这份报告是用哪个模型、哪版 prompt、什么参数生成的」。

### 1.2 明确非目标（v0 不做）

- 不做论文全文数据库、不做 PDF 全文抓取（只处理开放获取可合法获取的文本）。
- 不做引用网络/引文图分析、不做系统性综述（PRISMA）级别的方法学。
- 不做网页 UI、不做多用户/账号体系；v0 只有 CLI。
- 不为了「更全」而接入需要付费或违反 tos 的数据源。

---

## 2. 架构级决策（改动需谨慎）

以下决策属于架构级，修改必须在同一次变更中更新本文件并记录到 `docs/decisions/`。

1. **编排形态**：确定性流水线（pipeline）+ 分析阶段 fan-out 并行。不引入通用 agent 框架（LangGraph/AutoGen 等）作为 v0 依赖；直接用一个薄编排器（`orchestrator.py`）实现，便于测试与审计。
2. **模型供应商**：DeepSeek，OpenAI 兼容接口。业务代码只依赖内部 `LLMClient` 协议，不直接依赖任何 SDK。
3. **结构化契约**：Agent 之间只传递 `src/briefing/schemas.py` 中的 pydantic 模型，**禁止裸 `dict` 跨 Agent 传递**。
4. **可复现**：所有外部 HTTP 响应可缓存；缓存命中必须体现在 `manifest.json` 中。
5. **引用可验证**：引文只能来自检索阶段真实取回的元数据，LLM 不得生成参考文献条目。

---

## 3. 多 Agent 设计

每个 Agent 是一个可独立测试的纯函数/类：`async def run(input_model) -> output_model`。

| Agent | 职责 | 输入 | 输出 | LLM? |
| --- | --- | --- | --- | --- |
| `Planner` | 把 topic 拆成查询词、同义词、时间窗、纳入/排除标准 | `TopicRequest` | `SearchPlan` | 是 |
| `Retriever` | 调外部数据源检索候选（目标 20–40 篇） | `SearchPlan` | `CandidateList` | 否 |
| `Screener` | 去重、过滤、按相关性排序，选出 3–5 篇 | `CandidateList` | `SelectedPapers` | 是 |
| `Analyzer` | **每篇并行**：抽取问题/方法/数据/结论/局限/可复用点 | `Paper` | `PaperAnalysis` | 是 |
| `Synthesizer` | 横向对比、共性、分歧、研究空白、开放问题 | `SelectedPapers` + `[PaperAnalysis]` | `Briefing` | 是 |
| `Verifier` | 校验每条引用存在于 `papers.json`，标注无支撑断言 | `Briefing` + `SelectedPapers` | `VerificationReport` | 是（可选规则） |
| `Renderer` | `Briefing` → HTML → PDF | `Briefing` | `report.pdf` | 否 |

编排规则：

- `Analyzer` 阶段并行执行，并发数受 `MAX_CONCURRENCY`（默认 4）限制。
- 单个论文分析失败：重试 ≤2 次；仍失败则**整次运行失败**，不得静默丢弃该论文（否则会出现「3–5 篇」中实际只有 2 篇被分析的情况）。
- `Verifier` 发现无支撑引用：先让 `Synthesizer` 修订一次；仍不通过则失败并输出 `verification_failed`，保留中间产物。
- 每个 Agent 的 prompt 独立存放于 `src/briefing/prompts/<agent>_v<N>.md`，**prompt 文件名带版本号**，版本写入 `manifest.json`。

---

## 4. 技术栈与依赖约束

- Python **3.11+**，依赖与虚拟环境统一用 `uv` 管理（`pyproject.toml` + `uv.lock`，lock 文件必须提交）。
- 运行期依赖限定在：`httpx`、`pydantic`(v2)、`pydantic-settings`、`tenacity`、`jinja2`、`weasyprint`、`pypdf`。新增运行期依赖需在 `docs/decisions/` 里说明理由。
- 开发依赖：`pytest`、`pytest-asyncio`、`respx`、`ruff`、`mypy`。
- 全链路 `async` I/O；禁止在异步代码里用阻塞式 `requests`/`time.sleep`。
- 除 PDF 渲染外不得使用线程池/多进程。

---

## 5. 目录结构（约定，不得随意散落文件）

```
.
├── AGENTS.md                  # 本文件
├── README.md                  # 面向使用者的快速上手
├── pyproject.toml
├── uv.lock
├── .env.example               # 与 .env 保持同步，只含占位值
├── .githooks/pre-commit       # 拒绝提交密钥（每个 clone 启用一次）
├── TASK.md                    # 分步执行计划
├── docs/
│   ├── SPEC.md                # 当前范围、进度、待办
│   ├── architecture.md        # 多 Agent 架构、契约、状态机、不变量
│   └── decisions/NNN-*.md     # 关键决策记录
├── src/briefing/
│   ├── cli.py                 # 唯一 CLI 入口（stdlib argparse）
│   ├── orchestrator.py        # 流水线编排
│   ├── config.py              # pydantic-settings 配置
│   ├── schemas.py             # 所有 Agent 契约模型
│   ├── manifest.py            # 运行审计契约 + 阶段检查点/指纹
│   ├── evidence.py            # 引文溯源规则（无 LLM 依赖，供分析与验证共用）
│   ├── rendering.py           # 论文目录渲染（合成与审计共用同一份材料）
│   ├── agents/                # planner/retriever/normalizer/screener/analyzer/synthesizer
│   ├── llm/                   # deepseek_client.py / stub_client.py / budget.py + 协议
│   ├── sources/               # arxiv.py / replay.py / cache.py + Source 协议
│   ├── verify/                # deterministic.py(L1) / semantic.py(L2) / report.py
│   ├── report/                # references.py / markdown.py / briefing_md.py / render_pdf.py / templates
│   └── prompts/               # 带版本号的 prompt
├── tests/
│   ├── unit/                  # 全离线
│   ├── integration/           # 离线端到端 / PDF 渲染 / live 冒烟
│   └── fixtures/              # 录制的 HTTP 响应与模型回复
├── examples/                  # 人工确认过的样例产物
├── data/cache/                # 原始响应缓存（gitignore）
└── outputs/                   # 运行产物（gitignore；样例放 examples/）
```

根目录**不新增**临时脚本、notebook、草稿文件；一次性实验放在 `scratch/`（gitignore）或删除。

---

## 6. DeepSeek 接入规则

- 只通过 `src/briefing/llm/deepseek_client.py` 调用；接口为 OpenAI 兼容的 `POST {base_url}/chat/completions`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhoushy93/multi-agent-literature-briefing](https://github.com/zhoushy93/multi-agent-literature-briefing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
