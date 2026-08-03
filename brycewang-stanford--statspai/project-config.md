---
trigger: always_on
description: > Claude Code / AI agent 在本仓库工作的指引。动手前请通读。
---

# CLAUDE.md — StatsPAI

> Claude Code / AI agent 在本仓库工作的指引。动手前请通读。

---

## 1. 项目定位

**StatsPAI** 的目标是**面向 Agent 设计，适合人类以及 agent 进行调用，并致力于超越 Stata、R、以及老的 Python 生态，成为全世界最好的因果推断与实证分析工具**。

做法有三条：

1. **一次 `import statspai as sp`** 覆盖 DiD / IV / RD / 合成控制 / DML / Meta-Learner / 贝叶斯因果 / 因果发现 / 结构计量 / 面板 / 空间 / 时序。
2. **Agent 原生**——所有函数返回结构化结果、带自描述 schema，人和 Agent 用同一入口。**v1.6 起进入 P1 阶段**：`sp.causal_question`（estimand-first DSL）、`sp.llm_dag_propose / validate / constrained`（LLM-DAG 闭环）、`sp.paper()`（自动论文）、`sp.causal_text`（文本因果 MVP）。
3. **数值对齐 Stata / R**——已有参考实现的方法，先对齐再扩展。

| | |
| --- | --- |
| 版本 | `1.20.0`（见 [`pyproject.toml`](pyproject.toml)） |
| Python | 3.9 – 3.13 |
| License | MIT |
| 作者 | Biaoyue (Bryce) Wang · <brycew6m@stanford.edu> · CoPaper.AI / Stanford REAP |
| PyPI | <https://pypi.org/project/StatsPAI/> |
| 导入别名 | `import statspai as sp` —— 所有示例、docstring、测试一律 `sp.xxx` |

---

## 2. 仓库结构

```text
StatsPAI/
├── src/statspai/          # 主包：87 子模块 / 1,139 函数（实时数 `python scripts/registry_stats.py`）
│   ├── __init__.py          # 对外 API 入口
│   ├── registry.py          # 函数注册表（sp.help / sp.list_functions 依赖）
│   ├── help.py              # sp.help / sp.describe_function / sp.function_schema
│   ├── cli.py
│   └── <领域模块>/          # did iv rd synth dml metalearners …
├── rust/statspai_hdfe/    # HDFE Rust 后端（PyO3）
├── tests/                 # pytest 套件 + reference_parity/ + external_parity/
├── docs/ · mkdocs.yml     # MkDocs 文档
├── paper.md · paper.bib   # JOSS
├── benchmarks/            # 性能基准
└── pyproject.toml
```

领域分组（对外 API 按下列七类组织）：

- **因果 / 处理效应**：`causal did rd iv synth dml metalearners tmle bcf bayes causal_impact policy_learning ope dtr multi_treatment qte principal_strat proximal mediation mendelian assimilation bridge`
- **面板 / 结构**：`panel fixest structural frontier multilevel gformula gmm msm longitudinal`
- **空间 / 时序**：`spatial timeseries bartik`
- **因果发现 / ML**：`causal_discovery dag neural_causal deepiv conformal_causal matrix_completion bunching causal_llm causal_rl causal_text fairness`
- **设计 / 抽样 / 推断**：`matching power mht survey bounds dose_response interference selection censoring imputation transport target_trial epi survival surrogate`
- **分解 / 诊断 / 回归**：`decomposition regression nonparametric diagnostics robustness postestimation inference smart`
- **基础设施**：`core utils compat fast datasets output plots workflow agent experimental question`

---

## 3. 设计原则

1. **一次 import，统一 API**。能力通过 `sp.<function>` 暴露，不需要二级 import。
2. **Agent 原生**。`sp.list_functions()` / `sp.describe_function()` / `sp.function_schema()` 必须对所有对外符号有效。
3. **统一结果对象**。优先 `CausalResult`（或领域结果类），实现 `.summary()` `.plot()` `.to_latex()` `.to_word()` `.to_excel()` `.cite()`。
4. **家族方法用 dispatcher**。`sp.synth(method=...)` / `sp.decompose(method=...)` / `sp.dml(model=...)`——一个入口，多种估计器。
5. **先对齐 Stata / R**。`fixest` / `did` / `rdrobust` / `gsynth` / `MatchIt` / Stata 已有的，先对齐 API 和数值再扩展。
6. **证据优先**。数值正确性是底线，每个估计器必须有参考对齐或解析测试。
7. **失败要响亮**。假设违背 → 抛异常或 `warnings.warn` + 写入结果 `diagnostics`；不吞异常返回 `None` / `NaN`。orchestration / best-effort 路径（`workflow/` `smart/` `paper`）catch `Exception` 时**必须**调用 `statspai.workflow._degradation.record_degradation(target, section=..., exc=..., detail=...)`——发 `WorkflowDegradedWarning` + 把 `{section, error_type, message}` 追加到 `target.degradations`。**禁止** bare `except Exception: pass`——静默降级是隐藏正确性回归的最便宜方式。
8. **弃用走流程**。`DeprecationWarning` + [`MIGRATION.md`](MIGRATION.md) 登记 + 至少一个小版本缓冲期。
9. **引用零幻觉**。任何文献引用必须现场核验 DOI / 作者 / 年份 / 期刊，**禁止凭 LLM 记忆补全**。详见 §10《引用与文献》——这是 StatsPAI 的红线。

---

## 4. 代码规范

### 对外 API

- 新对外函数**必须注册** → [`src/statspai/registry.py`](src/statspai/registry.py)，否则 `sp.help` / `sp.list_functions` 看不到。
- docstring 用 NumPy 风格，包含 `Parameters` / `Returns` / `Examples` / `References`。
- `References` 段只写 **bib key**（对应 [`paper.bib`](paper.bib)）或**经过核验的规范引用**——禁止在 docstring 里手写未核验的引用字符串，详见 §10。
- 示例一律 `import statspai as sp` + `sp.xxx`。
- 破坏性改动 → [`MIGRATION.md`](MIGRATION.md) + `DeprecationWarning`。

### 模块内部

- 共享基元放模块级 `_core.py` / `_common.py`（参考 `rd/_core.py`、`decomposition/_common.py`）。**不要**在多个文件重复实现 kernel / WLS / sandwich / 影响函数。
- 私有函数 `_` 前缀。
- 单文件 ~800 行以内，按关注点拆（estimator / inference / diagnostics / plots），不是单纯按行数拆。

### 依赖

- 核心依赖精简（见 `pyproject.toml`）。重依赖归入 extras：`dev` / `performance` (jax) / `bayes` (pymc) / `neural` `deepiv` (torch) / `fixest` (pyfixest) / `plotting`。
- `torch` / `jax` / `pymc` 必须**惰性 import**，用户没装 extra 不应触发 `ImportError`。
- 禁止引入 GPL / AGPL 依赖（和 MIT 冲突）。

---

## 5. 测试

```bash
pytest                                        # 全量
pytest tests/test_did.py -q                   # 单文件
pytest -k bayes_iv                            # 关键字筛选
pytest --cov=statspai --cov-report=term-missing
pytest tests/reference_parity/ -q             # R/Stata 对齐
pytest tests/external_parity/ -q              # 论文数字对齐
```

### 新代码要求

- 每个对外函数：**正确性测试 + 边界测试**各至少一个。
- 新估计器：**有参考实现走对齐，没有走解析/仿真**，容差 `atol` / `rtol` 就地标注并说明理由。
- 核心估计器（`did iv rd synth dml panel`）目标覆盖率 **≥ 95%**；整仓目标 **≥ 85%**。
- **Windows CI 注意**：`Path.read_text()` 必须传 `encoding="utf-8"`（cp1252 默认会挂，见 commit `8755996`）。

---

## 6. Rust 组件

路径 [`rust/statspai_hdfe/`](rust/statspai_hdfe/)，HDFE 高性能后端，PyO3 打包，在 Python 侧通过 `sp.fast.*` / `sp.fixest.*` 暴露。

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brycewang-stanford/StatsPAI](https://github.com/brycewang-stanford/StatsPAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
