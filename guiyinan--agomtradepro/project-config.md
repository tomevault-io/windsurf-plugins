---
trigger: always_on
description: > 本文件是 AgomTradePro 项目的统一代理指引主文件，适用于 Codex、Claude Code 等代码代理。
---

# AGENTS.md - AgomTradePro 项目开发规则

> 本文件是 AgomTradePro 项目的统一代理指引主文件，适用于 Codex、Claude Code 等代码代理。
> 如存在其他代理说明文件，应以本文件为准，并链接回本文件，避免规则漂移。

## 项目概述

> **最后更新**: 2026-04-21
> **系统版本**: AgomTradePro 0.7.0
> **项目状态**: 生产就绪
> **业务模块**: 35个
> **MCP 工具**: 302个（本地注册快照）
> **测试规模**: 5,212 个已收集测试用例

AgomTradePro (Agom Strategic Asset Allocation Framework) 是个人投研平台，通过 Regime（增长/通胀象限）和 Policy（政策档位）过滤，确保投资者不在错误的宏观环境中下注。

**最新完成** (0.7.0):
- 新增 Setup Wizard 模块（系统初始化向导）
- 新增 AI Capability Catalog 模块（系统级 AI 能力目录与统一路由）
- 新增 Terminal CLI 模块（终端风格 AI 交互界面）
- 新增 Pulse 脉搏层模块（战术层脉搏指标聚合与转折预警）
- 完整的 DDD 四层架构实现

**版本管理**: 参见 [docs/VERSION.md](docs/VERSION.md)

## 技术栈

- Python 3.11+
- Django 5.x
- SQLite（开发）/ PostgreSQL（生产，后续开发）
- Celery + Redis（异步任务）
- Pandas + NumPy（数据处理）

## 核心架构约束 ⚠️

本项目严格遵循**四层架构**，违反以下规则的代码必须拒绝：

### Domain 层 (`apps/*/domain/`)
```
✅ 允许：Python 标准库、dataclasses、typing、enum、abc
❌ 禁止：django.*、pandas、numpy、requests、任何外部库
```
- 包含：entities.py（数据实体）、rules.py（业务规则）、services.py（纯算法）
- 所有金融逻辑必须在此层
- 使用 `@dataclass(frozen=True)` 定义值对象

### Application 层 (`apps/*/application/`)
```
✅ 允许：Domain 层、Protocol 接口
❌ 禁止：直接导入 ORM Model、直接调用外部 API
```
- 包含：use_cases.py（用例编排）、tasks.py（Celery 任务）、dtos.py
- 通过依赖注入使用 Infrastructure 层
- **新增/修改代码必须满足 CI 架构护栏**：
  - 不得新增 `from apps.*.infrastructure.models import ...`
  - 不得新增 `from ..infrastructure.models import ...`
  - 不得新增任何 `.objects.` / `Model.objects` 直接 ORM 访问
  - 不得在函数内部用延迟 import 绕过上述规则
- Application 层如需读写数据库，必须通过以下方式之一：
  - 调用本 App Infrastructure 层的 Repository / Adapter
  - 调用其他 App 已暴露的 Application UseCase / Service
  - 通过构造函数注入 Repository 接口，测试中用 fake/mock 替换

```python
# ❌ 错误：Application 层直接碰 ORM，会触发 Architecture Layer Guard
from apps.equity.infrastructure.models import StockInfoModel

def execute():
    return StockInfoModel.objects.filter(is_active=True)

# ✅ 正确：ORM 查询放在 Infrastructure Repository
class EquityReadRepository:
    def list_active_stocks(self):
        from apps.equity.infrastructure.models import StockInfoModel
        return StockInfoModel._default_manager.filter(is_active=True)

class SomeUseCase:
    def __init__(self, equity_repo: EquityReadRepository):
        self.equity_repo = equity_repo

    def execute(self):
        return self.equity_repo.list_active_stocks()
```

### Infrastructure 层 (`apps/*/infrastructure/`)
```
✅ 允许：Django ORM、Pandas、外部 API 客户端
```
- 包含：models.py（ORM）、repositories.py（数据仓储）、adapters/（API 适配器）
- 实现 Domain 层定义的 Protocol 接口

### Interface 层 (`apps/*/interface/`)
- 包含：views.py（DRF）、serializers.py、admin.py、urls.py
- 只做输入验证和输出格式化，禁止业务逻辑
- 不得新增 `from apps.*.infrastructure...`；Interface 必须调用 Application UseCase / Query Service。

### CI 架构护栏（开发前必须内化）

GitHub Actions 会对本次新增行做结构扫描。以下新增代码会直接失败：

```text
apps/*/domain/      禁止 import django / pandas / numpy / requests
apps/*/application/ 禁止 import *.infrastructure.models，禁止 .objects.
apps/*/interface/   禁止 import *.infrastructure.
```

开发时的判断原则：

- **Domain**：只放业务实体、值对象、纯规则，不知道 Django 存在。
- **Application**：只编排业务流程，不知道 ORM Model 存在。
- **Infrastructure**：负责 ORM、Pandas、外部 API、缓存、文件和网络 I/O。
- **Interface**：负责 HTTP/DRF 输入输出，不直接查数据库、不拼业务规则。
- 如果一个 Application 方法“只是查几张表组装上下文”，也必须把查表部分下沉到 Infrastructure Repository。
- 如果跨 App 需要数据，优先调用对方 Application UseCase；没有合适 UseCase 时，在拥有数据的 App Infrastructure 增加 Repository 方法，再通过依赖注入使用。

## 目录结构

```
AgomTradePro/
├── core/                     # Django 配置
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── celery.py
├── apps/                     # 35个业务模块
│   ├── macro/                # 宏观数据采集
│   ├── regime/               # Regime 判定引擎
│   ├── policy/               # 政策事件管理
│   ├── signal/               # 投资信号管理
│   ├── backtest/             # 回测引擎
│   ├── audit/                # 事后审计
│   ├── asset_analysis/       # 通用资产分析框架
│   ├── equity/               # 个股分析
│   ├── fund/                 # 基金分析
│   ├── sector/               # 板块分析
│   ├── sentiment/            # 舆情情感分析
│   ├── account/              # 账户与持仓管理
│   ├── simulated_trading/    # 模拟盘自动交易
│   ├── realtime/             # 实时价格监控
│   ├── data_center/          # 数据中台
│   ├── strategy/             # 策略系统
│   ├── ai_provider/          # AI 服务商管理
│   ├── prompt/               # AI Prompt 模板
│   ├── dashboard/            # 仪表盘
│   ├── filter/               # 筛选器管理
│   ├── alpha/                # AI 选股信号（Qlib 集成）
│   ├── alpha_trigger/        # Alpha 离散触发
│   ├── beta_gate/            # Beta 闸门
│   ├── decision_rhythm/      # 决策频率约束
│   ├── factor/               # 因子管理
│   ├── rotation/             # 板块轮动
│   ├── hedge/                # 对冲策略
│   ├── events/               # 事件系统
│   ├── terminal/             # 终端 CLI（AI 交互界面）
│   ├── agent_runtime/        # Agent 运行时（Terminal AI 后端）
│   ├── ai_capability/        # AI 能力目录（统一路由）
│   ├── pulse/                # Pulse 脉搏层（战术指标聚合与转折预警）
│   ├── share/                # 分享功能
│   ├── task_monitor/         # 任务监控
│   └── setup_wizard/         # 系统初始化向导
├── shared/                   # 跨 App 共享（仅技术性组件）
│   ├── domain/interfaces.py  # Protocol 定义
│   ├── infrastructure/       # 通用算法实现（如 Kalman 滤波）
│   └── config/secrets.py     # 密钥管理
└── tests/
```

## apps/ vs shared/ 架构边界 ⚠️

### apps/ - 业务模块（Business Modules）

**定义：** 拥有独立业务能力的完整四层架构模块

**必须放在 apps/ 的条件：**
1. ✅ 提供独立的业务能力（如"资产评分"、"回测计算"）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guiyinan/agomTradePro](https://github.com/guiyinan/agomTradePro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
