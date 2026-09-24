---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QuantLLM — A股量化交易系统，两条链路：

1. **多 Agent 决策链路（src/）**：LangGraph 编排 6 分析师 → 风控 → 组合经理最终裁决。决策层 = TypeSafe Jev 结构化决策优先，Qwen3.8-27B 兜底，规则 hold 最后兜底。
2. **训练链路（scripts/ + run.sh）**：数据采集 → 转换 → QLoRA 微调 → 评估 → GGUF 导出。

- 基座模型：Qwen3.8-27B（unsloth 4bit QLoRA，HF cache 路径）
- 推理服务：Xinference + vLLM（`http://127.0.0.1:9997/v1`，AWQ）
- 决策模型：TypeSafe Jev（外部 API，`https://api.typesafe.ai/v1/systemone`）
- 硬件：RTX A5000 24GB, CUDA 12.2
- 训练框架：Unsloth + TRL SFTTrainer + PEFT
- 项目目录：`/opt/quant-llm/`
- Python 环境：`/opt/quant-llm/finetune-env/`（venv, Python 3.12）
- 架构详情：`docs/architecture.md`

## Commands

所有操作通过 `run.sh` 入口执行，需先激活 venv：

```bash
source /opt/quant-llm/finetune-env/bin/activate

# 完整流水线
bash /opt/quant-llm/run.sh all

# 单步执行
bash /opt/quant-llm/run.sh crawl      # 爬取 A股+多市场数据
bash /opt/quant-llm/run.sh recalc     # 重算衍生指标
bash /opt/quant-llm/run.sh fund-flow  # 资金流数据
bash /opt/quant-llm/run.sh convert    # 市场数据 → ChatML 训练对
bash /opt/quant-llm/run.sh predict    # 预测性训练数据
bash /opt/quant-llm/run.sh generate   # 数据增强（FinGPT+量化计算+推理链）
bash /opt/quant-llm/run.sh factors    # 构建个股因子
bash /opt/quant-llm/run.sh merge      # 合并所有数据源 → merged_train_v4.jsonl
bash /opt/quant-llm/run.sh train      # QLoRA 微调
bash /opt/quant-llm/run.sh eval       # 评估（65题手写+holdout）
bash /opt/quant-llm/run.sh backtest   # 回测验证
bash /opt/quant-llm/run.sh rag-build  # 构建 RAG 检索索引
bash /opt/quant-llm/run.sh rag-serve  # 启动 RAG 增强推理服务
bash /opt/quant-llm/run.sh export     # 导出 GGUF
bash /opt/quant-llm/run.sh trade-live # 双层实盘决策（规则+Qwen/Jev）

# 多 Agent 决策（src/）
source /opt/quant-llm/finetune-env/bin/activate
python -m src.main --ticker 600519.SH                    # 单股全分析师
python -m src.main --list-analysts                       # 列出分析师
python -m src.main --ticker 600519.SH --show-reasoning   # 显示推理过程

# 直接运行单个脚本
python scripts/evaluate.py --consistency 3   # 带一致性检测的评估
```

## Architecture

### 数据流水线

```
crawl_ashare.py + crawl_multi_market.py    → training-data/{ashare,futures,etf,cbond}/*.jsonl
        ↓
convert_all_to_training.py                  → all_market_train.jsonl (~10k)
        ↓
fetch_fingpt_data.py                        → fingpt_forecaster.jsonl (1.2k)
generate_quant_calculations.py (Xinference) → quant_calculations.jsonl (~500)
add_reasoning_chains.py (Xinference)        → reasoning_enhanced.jsonl (~2k)
        ↓
merge_and_retrain.py                        → merged_train_v4.jsonl
        ↓
train.py                                    → output/（Qwen3.8-27B LoRA）
        ↓
evaluate.py                                 → output/eval_results_v{N}.json
export_gguf.py                              → output/gguf/
```

### 核心模块

- **`scripts/_config.py`** — 所有训练脚本的配置入口。加载 `config.yaml` + `.env`（`_load_env`，已存在环境变量优先），校验必要字段和取值范围，提供 `cfg` 字典、路径常量和 `call_ollama()` 辅助函数（实际打 Xinference OpenAI 兼容端点）。
- **`src/utils/llm.py`** — 多 Agent 链路的 LLM 调用（`call_llm` + Jev 的 `call_jev_choices`），顶部自动加载 `.env`（`_load_env_file`）。
- **`config.yaml`** — 中心化配置：数据路径、LoRA 参数（r=32, rslora）、训练超参、风控参数等。
- **`.env`** — 模型与凭据唯一来源（OLLAMA_URL / OLLAMA_*_MODEL / XINFERENCE_API_KEY / TYPESAFE_API_KEY / JEV_ENABLED），改模型/开关 Jev 只改 `.env`。参考 `.env.example`（.env 已 gitignore）。
- **`convert_all_to_training.py`** — 按市场类型生成领域特定问答模板（技术分析、交易信号 JSON、评分等），每个市场有独立的模板集。
- **`train.py`** — QLoRA 训练：4bit 量化加载 → LoRA 挂载（7个目标模块）→ ChatML 格式化 → 分层抽样 train/val split → SFTTrainer + early stopping + cosine annealing。
- **`evaluate.py`** — 三维评估：65 道手写题（含 15 道对抗性）+ holdout 集 + ROUGE-L/数值正确性/一致性指标。结果按版本存档。

### 多 Agent 决策链路（src/）

```
start → [6 分析师并行] → 风控管理 → 组合经理 → 最终决策
```

- **6 分析师**（`src/utils/analysts.py` 注册）：技术面、基本面、资金面、情绪面、估值、市场环境
- **风控**（`src/agents/risk_manager.py`）：波动率仓位上限 + 单票 10% + T+1
- **组合经理**（`src/agents/portfolio_manager.py`）：Jev 每只股票独立裁决（校准概率，<1s）→ 失败回落 Qwen → 规则 hold。`JEV_ENABLED=false` 切回纯 Qwen
- **关键经验**：Jev 批量调用会跨股票污染决策，必须每只独立调用；conf=0/neutral 信号进 Jev 前过滤
- 入口：`python -m src.main --ticker ...`；详情 `docs/architecture.md`

### RAG 检索增强（已实现）

```
用户查询 → bge-large-zh-v1.5 编码 → FAISS 检索 top-3 → 注入 system prompt → ollama 推理
```

- **向量库**：FAISS (faiss-cpu)，IndexFlatIP，~200MB 索引
- **Embedding**：BAAI/bge-large-zh-v1.5（1024维，CPU 运行避免抢 GPU）
- **索引内容**：merged_train_v3.jsonl 的 Q+A 对（只编码 user question）
- **不索引**：原始行情 OHLCV（推理时以 `[MARKET_DATA]` 结构化注入）
- **检索策略**：top_k=3，score_threshold=0.35，含 `[MARKET_DATA]` 的查询跳过 RAG
- **Prompt 注入**：参考资料放在 system prompt 中的 `[参考资料]...[/参考资料]` 块
- **Token 预算**：系统 50 + 参考资料 460 + 问题 200 = 710，剩余 1338 给生成
- **新增文件**：`scripts/rag_build_index.py`、`scripts/rag_retrieve.py`、`scripts/rag_serve.py`
- **配置**：config.yaml 新增 `rag:` 段（enabled/embedding_model/top_k/score_threshold 等）
- **不引入**：LangChain、ChromaDB、GPU embedding、re-ranking

### 关键设计决策

- **推理链（`<think>` 标签）只用于知识解释和量化计算类问题**，不用于交易决策（基于 StockBench 研究结论）
- **交易信号统一为 JSON 输出格式**：`{action, symbol, reason, confidence, stop_loss}`
- **数据增强依赖本地 ollama**：模型配置在 `.env`（当前 qwen3.8:27b），种子扩展/推理链共用，改模型只改 `.env`
- **训练数据带来源标记**（`source` 字段），支持分层抽样和来源分析

## Config Reference

训练关键参数在 `config.yaml` 中（不要硬编码）：

| 参数 | 值 | 说明 |
|------|-----|------|
| lora.r | 32 | LoRA 秩 |
| lora.use_rslora | true | Rank-Stabilized LoRA |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chl-5g/QuantLLM](https://github.com/chl-5g/QuantLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
