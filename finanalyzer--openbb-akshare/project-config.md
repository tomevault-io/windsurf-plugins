---
trigger: always_on
description: **openbb_akshare** 是 OpenBB Platform 的数据源扩展插件，将中国金融数据聚合库 AKShare 集成到 OpenBB 平台中。该项目解决了中国内地用户访问 OpenBB 时需要 VPN 的痛点，通过 AKShare 提供本地化的 A 股、港股等市场数据接口。
---

# openbb_akshare 项目上下文

## 项目概述

**openbb_akshare** 是 OpenBB Platform 的数据源扩展插件，将中国金融数据聚合库 AKShare 集成到 OpenBB 平台中。该项目解决了中国内地用户访问 OpenBB 时需要 VPN 的痛点，通过 AKShare 提供本地化的 A 股、港股等市场数据接口。

### 核心功能

- **股票数据**: A 股和港股的历史价格、实时报价、公司概况、股票搜索、股权结构等
- **财务数据**: 资产负债表、现金流量表、利润表、关键指标等
- **市场分析**: 公司新闻、价格表现、业务分析等
- **基金数据**: ETF 持仓、基金持仓、ETF 搜索等
- **货币数据**: 货币历史价格、货币快照等
- **指数数据**: 可用指数列表

### 技术栈

- **Python**: 3.11 - 3.12
- **核心依赖**:
  - `akshare` (1.18.19): 中国金融数据聚合库
  - `openbb` (4.6.0): OpenBB Platform 核心库
  - `openbb-core` (^1.5.8): OpenBB 核心接口
  - `mysharelib` (^1.0.4): 自定义工具库
- **开发依赖**:
  - `pytest` (9.0.1): 测试框架
  - `ipykernel` (6.30.1): Jupyter 内核
  - `uvicorn` (0.40.0): ASGI 服务器

### 项目架构

```
openbb_akshare/
├── openbb_akshare/          # 主包目录
│   ├── __init__.py          # Provider 定义和导出
│   ├── openbb.py            # 应用工厂和扩展构建
│   ├── router.py            # 路由命令定义
│   ├── models/              # 数据获取器 (Fetchers)
│   │   ├── available_indices.py
│   │   ├── balance_sheet.py
│   │   ├── business_analysis.py
│   │   ├── cash_flow.py
│   │   ├── company_news.py
│   │   ├── currency_historical.py
│   │   ├── currency_snapshots.py
│   │   ├── equity_historical.py
│   │   ├── equity_ownership.py
│   │   ├── equity_profile.py
│   │   ├── equity_quote.py
│   │   ├── equity_screener.py
│   │   ├── equity_search.py
│   │   ├── etf_holdings.py
│   │   ├── etf_search.py
│   │   ├── fund_holdings.py
│   │   ├── historical_dividends.py
│   │   ├── income_statement.py
│   │   ├── key_metrics.py
│   │   └── price_performance.py
│   ├── standard_models/     # 标准模型定义
│   │   ├── business_analysis.py
│   │   └── fund_holdings.py
│   └── utils/               # 工具函数和辅助模块
│       ├── ak_balance_sheet.py
│       ├── ak_cash_flow.py
│       ├── ak_compare_company_facts.py
│       ├── ak_equity_ownership.py
│       ├── ak_equity_search.py
│       ├── ak_income_statement.py
│       ├── ak_key_metrics.py
│       ├── fetch_equity_info.py
│       ├── fetch_quote.py
│       ├── helpers.py
│       └── references.py
├── tests/                   # 测试文件
│   ├── conftest.py          # Pytest 配置和 fixtures
│   ├── test_*.py            # 各功能模块测试
│   └── debug_*.py           # 调试脚本
└── docs/                    # 文档和图片
```

## 构建和运行

### 环境设置

项目使用 Poetry 进行依赖管理，但也可以使用 pip 安装。

**方法 1: 使用 Poetry**

```bash
# 安装依赖
poetry install

# 激活虚拟环境
poetry shell
```

**方法 2: 使用 pip**

```bash
# 创建虚拟环境
python -m venv .venv

# 激活虚拟环境 (Windows)
.venv\Scripts\activate

# 激活虚拟环境 (Linux/Mac)
source .venv/bin/activate

# 安装依赖
pip install -e .
```

### 构建项目

```bash
# 构建所有模块
python -c "from openbb_akshare import build; build()"

# 或者使用 OpenBB 的构建命令
python -c "import openbb; openbb.build()"
```

### 运行测试

```bash
# 运行所有测试
pytest

# 运行特定测试文件
pytest test_company_news.py

# 运行特定测试函数
pytest test_company_news.py::test_company_news_fetcher

# 显示详细输出
pytest -v

# 显示打印输出
pytest -s
```

### 使用 OpenBB CLI

```bash
# 启动 OpenBB CLI
openbb

# 在 CLI 中使用 akshare 提供商
/news/company --symbol 000002 --provider akshare
/equity/price/historical --symbol 06823 --provider akshare
```

### 在 Python 代码中使用

```python
from openbb import obb

# 获取公司新闻
news = obb.news.company("000002", provider="akshare")

# 获取历史股价
prices = obb.equity.price.historical(
    symbol="06823",
    start_date="2025-06-01",
    end_date="2025-06-10",
    provider="akshare"
)

# 获取财务数据
balance_sheet = obb.equity.fundamental.balance_sheet("000002", provider="akshare")
```

## 开发约定

### 代码风格

- **类型提示**: 使用 Python 类型提示，特别是在函数签名中
- **文档字符串**: 使用 Google 风格的 docstring
- **导入顺序**: 遵循 PEP 8 标准导入顺序
- **命名约定**:
  - Fetcher 类: `AKShare<Name>Fetcher` 或 `Akshare<Name>Fetcher`
  - 模块函数: 使用 snake_case
  - 常量: 使用 UPPER_CASE

### Fetcher 实现

每个数据获取器 (Fetcher) 需要遵循以下模式：

```python
from openbb_core.provider.abstract.fetcher import Fetcher
from openbb_core.provider.standard_models.<model_name> import <ModelName>QueryParams

class AKShare<Name>Fetcher(Fetcher[<ModelName>QueryParams, List[<ModelName>Data]]):
    @staticmethod
    def transform_query(params: Dict[str, Any]) -> <ModelName>QueryParams:
        """转换查询参数。"""
        # 转换逻辑
        return params

    @staticmethod
    def extract_data(query: <ModelName>QueryParams, credentials: Optional[Dict[str, str]], **kwargs) -> List[Dict]:
        """从 AKShare 获取原始数据。"""
        # 调用 akshare 函数
        return raw_data

    @staticmethod
    def transform_data(data: List[Dict], **kwargs) -> List[<ModelName>Data]:
        """转换数据为标准格式。"""
        # 数据转换逻辑
        return transformed_data
```

### Provider 注册

在 `openbb_akshare/__init__.py` 中注册所有 Fetcher：

```python
provider = Provider(
    name="akshare",
    description="Data provider for openbb-akshare.",
    credentials=["api_key"],
    website="https://akshare.akfamily.xyz/",
    fetcher_dict={
        "FetcherName": AKShareFetcherName,
        # ... 其他 fetchers
    }
)
```

### 环境变量

- `AKSHARE_API_KEY`: AKShare API 密钥（某些功能可能需要）
- `PIP_INDEX_URL`: pip 镜像地址（可选，用于国内加速）

### 测试约定

- 使用 pytest 作为测试框架
- 测试文件命名: `test_*.py`
- 使用 fixtures 提供共享资源（在 `conftest.py` 中定义）
- 每个 Fetcher 应有对应的测试文件
- 调试脚本命名: `debug_*.py`

### 工具函数

工具函数位于 `openbb_akshare/utils/` 目录：

- `helpers.py`: 通用辅助函数
- `fetch_quote.py`: 股票报价获取
- `fetch_equity_info.py`: 股票信息获取

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finanalyzer/openbb_akshare](https://github.com/finanalyzer/openbb_akshare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
