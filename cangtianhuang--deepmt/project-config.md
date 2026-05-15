---
trigger: always_on
description: 本文件为 Claude Code 提供项目上下文指引。
---

# CLAUDE.md

本文件为 Claude Code 提供项目上下文指引。

## 项目简介

DeepMT（Deep Metamorphic Testing）是面向深度学习框架（PyTorch、TensorFlow、PaddlePaddle）的**蜕变关系（MR）自动生成与分层测试系统**。

## 当前阶段与下一步

> **在开始任何”阶段开发”任务前，必须先阅读 `docs/dev/` 中的规划文档。”代码修复”任务不必查阅。**  
> 入口：`docs/dev/status.md` → 对应阶段文档。  
> 执行规范：`docs/dev/agent_rules.md`

| 阶段                                  | 状态     | 文档                                                           |
| ------------------------------------- | -------- | -------------------------------------------------------------- |
| Phase A：算子数据层完善               | ✅ 完成   | `docs/dev/achived/01_Phase_A_算子数据层完善.md`                |
| Phase B：算子层 MR 生成与知识库       | ✅ 完成   | `docs/dev/achived/02_Phase_B_算子层MR生成与知识库.md`          |
| Phase C：测试执行与跨框架适配         | ✅ 完成   | `docs/dev/achived/03_Phase_C_测试执行与跨框架适配.md`          |
| Phase D：缺陷分析与实验闭环           | ✅ 完成   | `docs/dev/achived/04_Phase_D_缺陷分析、实验闭环与研究结论.md`  |
| Phase E：演示交付与生产化加固         | ✅ 完成   | `docs/dev/achived/05_Phase_E_演示交付与生产化加固.md`          |
| Phase F：软件工程规范化               | ✅ 完成   | `docs/dev/achived/06_Phase_F_软件工程规范化与包发布准备.md`    |
| Phase G：统一IR与三层对象建模         | ✅ 完成   | `docs/dev/archived/07_Phase_G_统一IR与三层对象建模.md`         |
| Phase H：第二框架落地与真实跨框架适配 | ✅ 完成   | `docs/dev/archived/08_Phase_H_第二框架落地与真实跨框架适配.md` |
| Phase I：模型层MR自动生成引擎         | ✅ 完成   | `docs/dev/archived/09_Phase_I_模型层MR自动生成引擎.md`         |
| Phase J：应用层语义MR生成与验证       | ✅ 完成   | `docs/dev/archived/10_Phase_J_应用层语义MR生成与验证.md`       |
| Phase K：全层MR质量保障与知识库治理   | ✅ 完成   | `docs/dev/archived/11_Phase_K_全层MR质量保障与统一知识库治理.md`        |
| Phase L：论文实验基准与自动化数据生产 | ✅ 完成   | `docs/dev/archived/12_Phase_L_论文实验基准与自动化数据生产线.md`        |
| Phase M：真实缺陷挖掘与案例沉淀       | 🔄 进行中 | `docs/dev/13_Phase_M_真实缺陷挖掘与案例沉淀.md`                         |
| Phase M 系统能力缺口修复（T1~T9）     | ✅ 完成   | `docs/dev/archived/15_Phase_M_system_capability_gaps.md`                |
| Phase N：论文交付收口与复现资产封装   | ⬜ 未开始 | `docs/dev/14_Phase_N_论文交付收口与复现资产封装.md`                      |
| Phase O：核心框架插件闭环与健康管理   | ✅ 完成   | `docs/dev/archived/16_Phase_O_framework_plugin_closure_and_health.md`   |
| Phase P：仪表盘三层重设计             | ✅ 完成   | `docs/dev/archived/17_Phase_P_仪表盘三层重设计.md`                      |

**当前主链：** A~O + Phase P 均完成 → **当前进行：Phase M 真实缺陷挖掘主干**（用户手动执行）→ Phase N 论文交付收口未开始

## 环境与运行

项目使用 `uv` 管理虚拟环境：

```bash
source .venv/bin/activate && PYTHONPATH=$(pwd) python -m pytest tests/
```

关键环境变量：`OPENAI_API_KEY`、`DEEPMT_LOG_LEVEL`、`DEEPMT_LOG_DIR`。详见 `docs/environment_variables.md`。

## 文档

| 文件                            | 内容               |
| ------------------------------- | ------------------ |
| `docs/dev/status.md`            | 已完成模块清单     |
| `docs/dev/agent_rules.md`       | 编码智能体执行规范 |
| `docs/dev/`                     | 各阶段规划文档     |
| `docs/cli_reference.md`         | CLI 命令参考       |
| `docs/environment_variables.md` | 环境变量说明       |
| `docs/tech/operator_catalog.md` | 算子目录设计       |
| `docs/tech/operator_mr.md`      | 算子层 MR 技术细节 |
| `docs/quick_start.md`           | 快速上手           |

## 架构概览

采用**微内核 + 插件化**架构。MR 生成与测试执行分离：先生成 MR 存入知识库（SQLite），再复用于多次测试。

### MR 生成四阶段流水线（`deepmt/mr_generator/operator/operator_mr_generator.py`）

1. **信息准备** — 提取算子代码与文档（网络搜索）
2. **候选生成** — LLM 猜想 + 模板池匹配
3. **快速筛选（Pre-check）** — 随机输入数值验证
4. **形式化验证** — SymPy 符号证明

### 核心数据结构（`deepmt/ir/schema.py`）

- `OperatorIR`：算子描述（名称、输入、输出、属性）
- `ModelIR`：模型描述（model_type、task_type、input_shape、model_instance 等）
- `ApplicationIR`：应用描述（task_type、domain、sample_inputs、sample_labels 等）
- `MetamorphicRelation`：MR 对象，包含 `transform_code`（输入变换 lambda）、`oracle_expr`（输出关系表达式）、`layer`（算子/模型/应用）、`lifecycle_state`、`source`

### 三层 MR 生成概览

| 层次   | 生成器                                     | 特点                                            |
| ------ | ------------------------------------------ | ----------------------------------------------- |
| 算子层 | `OperatorMRGenerator`（四阶段流水线）      | LLM 猜想 + 模板池 + 数值预检 + SymPy 证明       |
| 模型层 | `ModelMRGenerator`（模板驱动）             | 结构分析（GraphAnalyzer）→ 策略库选择 → 生成 MR |
| 应用层 | `ApplicationMRGenerator`（LLM + 模板回退） | 场景注册 → 上下文构建 → LLM/模板生成 → 语义验证 |

## 项目地图

```
.
├── deepmt/                 # 主包（CLI 入口 + 所有核心模块）
│   ├── __init__.py         #   公共 API（导出 DeepMT, TestResult）
│   ├── __main__.py         #   python -m deepmt 入口
│   ├── cli.py              #   CLI 命令组
│   ├── client.py           #   DeepMT / TestResult 高层 API
│   ├── commands/           #   CLI 子命令实现（mr / test/ / repo / catalog / data / health）
│   │   └── test/           #     test 命令包（execution / analysis / evidence / history）
│   ├── core/               #   微内核框架
│   │   ├── config_manager.py   #   配置加载与管理
│   │   ├── logger.py           #   日志（get_logger / log_structured）
│   │   ├── plugins_manager.py  #   插件加载
│   │   └── results_manager.py  #   结果管理
│   ├── engine/             #   测试执行引擎
│   │   ├── batch_test_runner.py  #   算子层批量测试执行器
│   │   └── model_test_runner.py  #   模型层测试执行器
│   ├── ir/                 #   统一中间表示
│   │   └── schema.py           #   三层 IR（OperatorIR/ModelIR/ApplicationIR）+ MR 数据结构
│   ├── mr_generator/       #   MR 生成引擎
│   │   ├── operator/           #   算子层（四阶段流水线）
│   │   │   ├── operator_mr_generator.py  #   主生成器
│   │   │   ├── operator_llm_mr_generator.py  # LLM 生成
│   │   │   ├── sympy_prover.py               # 符号证明

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cangtianhuang/DeepMT](https://github.com/cangtianhuang/DeepMT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
