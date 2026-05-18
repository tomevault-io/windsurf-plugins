---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

miniQMT 是一个基于迅投QMT API的**无人值守量化交易系统**,实现自动化交易策略执行、持仓管理、止盈止损和网格交易。

**核心特性**:
- 🔄 双层存储架构(内存数据库 + SQLite持久化)
- 🎯 信号检测与执行分离设计
- 🧵 多线程协同工作 + 线程自愈机制
- 📈 动态止盈止损策略
- 🌐 Web前端实时监控界面
- 🛡️ 无人值守运行(线程监控、超时保护、优雅关闭)

**环境要求**:
- Python 3.8+ (推荐 3.9)，例如用户目录下的Anaconda3/envs/python39
- 操作系统: Windows (QMT仅支持Windows)
- QMT客户端: 实盘交易需要安装并登录QMT

**依赖安装**:
```bash
pip install -r utils/requirements.txt
```

## ⚠️ 关键约束 - 违反将导致系统故障

**执行任何代码修改前必须遵守**:

1. **配置集中管理**: 所有可配置参数在 [config.py](config.py) 中,严禁硬编码魔法数字
2. **模拟交易优先**: 测试新功能前必须设置 `ENABLE_SIMULATION_MODE = True`
3. **线程安全**: 修改共享数据必须使用 `threading.Lock()` 保护
4. **信号验证**: 交易信号必须经过 `validate_trading_signal()` 验证,防止重复执行
5. **双层存储同步**: 修改内存数据库后必须调用 `_increment_data_version()`
6. **线程注册规范**: 注册线程监控时必须使用 `lambda` 获取线程对象(见下文)
7. **Git操作**: 除非用户明确要求,不要主动执行git提交和分支操作

## 快速开始

### 环境准备(推荐)
```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# 安装依赖
pip install -r utils/requirements.txt

# 验证安装
python utils/check_dependencies.py
```

### 配置文件
创建 `account_config.json` 和 `stock_pool.json` (参见文档末尾)

### 启动系统
```bash
python main.py
```
**首次运行**: 系统会自动创建 `data/positions.db` 数据库文件

### 运行测试

#### 回归测试框架 (推荐)

项目集成了完整的回归测试框架 ([test/run_integration_regression_tests.py](test/run_integration_regression_tests.py))，支持按模块运行、快速验证和失败重试。测试组配置在 [test/integration_test_config.json](test/integration_test_config.json)。

```bash
# 快速验证（5分钟内完成，检查关键功能）
python test/run_integration_regression_tests.py --fast

# 运行所有回归测试
python test/run_integration_regression_tests.py --all

# 按组运行
python test/run_integration_regression_tests.py --group system_integration  # 系统集成
python test/run_integration_regression_tests.py --group stop_profit         # 止盈止损
python test/run_integration_regression_tests.py --group grid_signal         # 网格信号
python test/run_integration_regression_tests.py --group grid_session        # 网格会话
python test/run_integration_regression_tests.py --group grid_trade          # 网格交易
python test/run_integration_regression_tests.py --group grid_exit           # 网格退出
python test/run_integration_regression_tests.py --group grid_validation     # 网格参数
python test/run_integration_regression_tests.py --group grid_comprehensive  # 网格综合
python test/run_integration_regression_tests.py --group grid_bug_regression # Bug回归验证
python test/run_integration_regression_tests.py --group grid_full_range_coverage  # 全区间覆盖

# 其他选项
python test/run_integration_regression_tests.py --all --retry-failed   # 失败重试
python test/run_integration_regression_tests.py --all --verbose        # 详细输出
python test/run_integration_regression_tests.py --all --skip-env-prep  # 跳过环境准备
python test/run_integration_regression_tests.py --all --no-backup      # 不备份生产DB
```

测试报告自动输出到 `test/integration_test_report.json` 和 `test/integration_test_report.md`。

#### 单个测试文件

```bash
# 运行单个测试模块
python test/run_single_test.py test.test_unattended_operation

# 直接使用 unittest
python -m unittest test.test_system_integration -v

# 运行全部网格测试
python test/run_all_grid_tests.py
```

### Web前端
浏览器访问: `http://localhost:5000`

### 系统诊断工具
```bash
# 检查系统状态
python test/check_system_status.py

# 诊断QMT连接
python test/diagnose_qmt_connection.py

# 查看实时日志
tail -f logs/qmt_trading.log  # Linux/Mac
Get-Content logs/qmt_trading.log -Wait  # Windows PowerShell
```

## 核心架构

### 关键设计原则

**1. 信号检测与执行分离** (最重要!)
```
持仓监控线程(始终运行) → 检测信号 → latest_signals队列
                                        ↓
策略执行线程 → 检查ENABLE_AUTO_TRADING → 执行/忽略信号
```

**关键点**:
- 监控线程**始终运行**,持续检测信号(即使 `ENABLE_AUTO_TRADING=False`)
- `ENABLE_AUTO_TRADING` 只控制**是否执行**检测到的信号
- 每个信号都要经过 `validate_trading_signal()` 验证,防止重复执行

**2. 双层存储架构**
```
实盘模式:
QMT实盘账户 → position_manager.qmt_trader.position() → 内存数据库
内存数据库 → 定时同步(15秒) → SQLite数据库

模拟模式:
Web界面 → trading_executor → position_manager.simulate_buy/sell() → 内存数据库
(跳过QMT接口,资金从SIMULATION_BALANCE扣除/增加)
```

**关键点**:
- 内存数据库存储高频更新数据(价格、市值、盈亏比例)
- SQLite持久化关键状态(开仓日期、止盈触发标记、最高价)
- 修改内存数据后必须调用 `_increment_data_version()` 触发前端更新

### 模块职责

```
config.py              # 集中配置管理(所有魔法数字都在这里)
logger.py              # 统一日志管理
main.py                # 系统启动入口和线程管理
thread_monitor.py      # 线程健康监控与自愈（无人值守核心）⭐
data_manager.py        # 历史数据获取(xtdata接口)
indicator_calculator.py # 技术指标计算
position_manager.py    # 持仓管理核心(内存+SQLite双层)⭐
trading_executor.py    # 交易执行器(xttrader接口)
strategy.py            # 交易策略逻辑⭐
web_server.py          # RESTful API服务(Flask)
easy_qmt_trader.py     # QMT交易API封装
premarket_sync.py      # 盘前同步与初始化(每天9:25重新初始化xtquant)
config_manager.py      # 配置持久化管理
sell_monitor.py        # 卖出委托单超时监控与撤单⭐
grid_trading_manager.py # 网格交易会话管理(独立线程)
grid_database.py       # 网格交易数据持久化(SQLite)
grid_validation.py     # 网格交易参数校验
xtquant_manager/       # XtQuantManager HTTP网关(多账户管理，可选)
```

### 线程架构

| 线程 | 启动位置 | 职责 | 频率 | 关键配置 |
|------|---------|------|------|---------|
| 线程监控 | `thread_monitor.start()` | 检测线程崩溃并自动重启 | 60秒 | `ENABLE_THREAD_MONITOR` |
| 数据更新 | `data_manager.start_data_update_thread()` | 更新股票池行情 | 60秒 | - |
| 持仓监控 | `position_manager.start_position_monitor_thread()` | 同步实盘持仓、更新价格、检测止盈止损 | 3秒 | `MONITOR_LOOP_INTERVAL` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weihong-su/miniQMT](https://github.com/weihong-su/miniQMT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
