---
trigger: always_on
description: > 适用范围：所有 Seagull 项目开发工作
---

# Seagull 量化研究平台 - 开发规范

> 适用范围：所有 Seagull 项目开发工作
> 版本：v1.3
> 更新日期：2026-05-19
> 专业等级：机构级 ■■■■■ | 深度：深度交叉验证 | 参考标准：MSCI Barra, Axioma Qontigo, Northfield, BlackRock Aladdin
>
> 作为MSCI Barra, Axioma Qontigo, Northfield, BlackRock Aladdin，Two-Sigma专业量化专家，你认为那种方式最合适最专业，容易管理和维护，也容易输出各种对比结果

---

## ⚠️ MANDATORY IMPORT RULE (HIGHEST PRIORITY)

**此规则覆盖所有其他代码模式，必须严格遵守：**

- ✅ **正确写法:** `from seagull.module import X`
- ❌ **禁止写法:** `from src.seagull.module import X`

**重要提示：**
- 即使你在代码库中看到 `from src.seagull.` 的旧模式，也 **不要** 复制或使用它
- 所有新代码和修改的代码 **必须** 使用 `from seagull.` 格式
- 违反此规则的提交将被自动拒绝
- 如果你发现还有文件使用错误格式，请先修复后再提交

---

## ⚠️ MANDATORY TECH STACK RULE (HIGH PRIORITY)

**此规则覆盖所有数据处理代码，必须严格遵守：**

- ✅ **首选技术栈：Polars + Parquet**
  - 所有新的 ETL、因子计算、数据处理代码 **必须** 使用 Polars 作为计算引擎
  - 所有持久化数据存储 **必须** 使用 Parquet 格式（列式存储 + 压缩）
  - 性能目标：比 Pandas + CSV 架构提升 **10-20 倍**

- ❌ **禁止默认使用 Pandas + CSV**
  - 除非有明确的兼容性理由（且需在 PR 中说明），否则禁止新代码使用 Pandas + CSV
  - 现有 Pandas 代码应逐步迁移到 Polars

### Polars 最佳实践（强制）

| 规则 | 原因 |
|------|------|
| ✅ 优先使用惰性执行 (`scan_parquet`) | 支持谓词下推、投影下推，全链路优化 |
| ✅ 使用 `over()` 做分组窗口计算 | 原生支持分组窗口，比 Pandas `groupby + join` 简洁 10 倍 |
| ✅ 表达式链式调用，避免中间变量 | 优化器可以做全局优化 |
| ✅ 优先使用内置 `rolling_*` 函数 | 向量化实现，比自定义 UDF 快 100x |
| ❌ 不要逐行 `apply` | 这是 Pandas 的坏习惯，Polars 中慢 100 倍 |
| ❌ 不要频繁 `to_pandas()` | 会触发完整内存复制，尽量全链路 Polars |

### Parquet 最佳实践（强制）

| 规则 | 原因 |
|------|------|
| ✅ Row Group 大小设为 50-100 万行 | 太小元数据开销大，太大谓词下推效果差 |
| ✅ 按日期分区存储 | `factor.parquet/date=2020-01-01/file.parquet`，按日期查询跳过整个目录 |
| ✅ 热数据用 Snappy 压缩，冷数据用 ZSTD | Snappy 速度快，ZSTD 压缩比高 30% |
| ✅ 按查询模式排序 | 经常按日期查询就按 date 排序，经常按股票查询就按 stock 排序 |
| ❌ 不要存成一个超大文件 | 最好控制在 100-500MB / 文件，利于并行读取 |
| ❌ 不要频繁小批量追加 | Parquet 不可变，追加 = 重写，用 Delta Lake 解决 |

### 标准工作流示例

```python
import polars as pl

# ✅ 惰性扫描 + 谓词/投影下推（关键优化）
result = (
    pl.scan_parquet("market_data.parquet")
      .select(["date", "stock", "close", "volume"])  # 只读需要的列
      .filter(pl.col("date") >= "2020-01-01")         # 只读需要的行
      .with_columns([
          pl.col("close").pct_change().over("stock").alias("return"),
          pl.col("close").rolling_mean(20).over("stock").alias("ma20"),
          (pl.col("close") / pl.col("close").rolling_mean(5).over("stock") - 1)
              .rank("average").over("date").alias("factor")
      ])
      .collect()  # 最终触发执行
)

# ✅ 结果写回 Parquet
result.write_parquet("factor_results.parquet", compression="snappy")
```

---

## 0. 项目结构与常用路径

### 0.1 目录结构

```
seagull/
├── src/seagull/              # 核心源码目录
│   ├── attribution/            # 归因与ablation研究 
│   ├── backtesting/            # 回测引擎
│   │   └── model/              # 回测模型
│   ├── core/                   # 核心工具与通用组件
│   ├── data/                   # 数据层
│   │   └── etl/                # ETL脚本
│   ├── execution/              # 交易执行
│   ├── factor/                 # 因子开发
│   └── risk/                   # 风险管理
├── tests/                      # 测试套件
├── docs/                       # 文档与设计
├── nas/doc/                    # 完整量化知识库（15个分类）
├── factor/                     # 因子相关文件
├── scripts/                    # 工具脚本
├── .venv/                      # 虚拟环境
├── pyproject.toml              # 项目配置
├── README.md                   # 项目文档
└── CLAUDE.md                   # 本文件 - 开发规范
```

### 0.2 常用路径速查表

| 路径 | 说明 | 典型使用场景 |
|------|------|--------------|
| `tests/` | 测试根目录 | 运行所有测试: `pytest tests/ -v` |
| `src/seagull/backtesting/model/` | 回测模型 | 添加新的策略模型、绩效计算 |
| `src/seagull/data/etl/` | 数据ETL | 新增数据源、数据清洗逻辑 |
| `src/seagull/factor/` | 因子开发 | 新因子实现、因子工具函数 |
| `src/seagull/attribution/` | 归因分析 | 收益归因、ablation研究 |
| `nas/doc/` | 完整知识库 | 架构文档、操作手册、SOP |
| `nas/doc/README.md` | **文档系统首选索引入口** | 查阅、整理、合并或新增文档前优先读取 |
| `nas/doc/GUIDE.md` | Seagull 文档编写指南 | 编写、重构、拆分、合并项目文档 |
| nas/doc/3_01_数据获取与处理/02-表结构/02-REF-表命名与分层枚举.md |  | ddl命名规范，data/etf文件命名规范 |

**Core 核心组件子目录：**

| 路径 | 说明 |
|------|------|
| `src/seagull/core/config/` | 配置管理、全局常量、环境配置 |
| `src/seagull/core/dag/` | Dagster 资产定义、Graph 定义 |
| `src/seagull/core/frontend/` | 前端页面、可视化组件 |
| `src/seagull/core/pipeline/` | Pipeline 基类、编排逻辑、状态管理 |
| `src/seagull/core/route/` | 路由配置、API 路由 |
| `src/seagull/core/services/` | 服务层、外部服务集成 |
| `src/seagull/core/utils/` | **最常用** - 通用工具函数、数学工具、日期处理 |
| `src/seagull/core/workflow/` | 工作流定义、任务调度 |
| `log/{module_name}.log` | **日志文件** - 每个模块独立日志，异常排查首选 |

### 0.2 Pipeline 数据 IO 统一入口

**推荐使用 `facade_data.py` 作为 Pipeline 数据读写统一入口：**

| 函数 | 说明 |
|------|------|
| `load_data` / `write_data` | DB/Parquet 统一读写 |
| `read_parquet_artifact` / `write_parquet_artifact` | Artifact 读写（Pipeline 中间结果） |
| `stable_artifact_hash` | 文件 SHA256 哈希（数据一致性验证） |
| `write_metadata_json` | 元数据 JSON 写入 |

```python
# ✅ 推荐：统一入口
from seagull.core.utils.facade_data import load_data, write_data, write_parquet_artifact

# 读取数据
df = load_data("dwd_stock_factor_preprocess_incr", input_source="parquet")

# 写入 Artifact
write_parquet_artifact(result_df, "output/result.parquet")
```

**兼容模式**：`facade_artifacts.py` 仍可使用，内部委托 `facade_data.py`。建议新代码直接使用 `facade_data.py`。

### 0.3 常用命令

```bash
# 运行测试
pytest tests/ -v
pytest tests/factor/test_turnover.py -v

# 运行 Pipeline
python src/seagull/main.py --pipeline factor --factor turnover

# Dagster UI
dagster dev -f src/seagull/main.py

# 代码格式化
black src/seagull/
isort src/seagull/

# 验证 Polars + Parquet 性能

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StarlightDamian/seagull](https://github.com/StarlightDamian/seagull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
