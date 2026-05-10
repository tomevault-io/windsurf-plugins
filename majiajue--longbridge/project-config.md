---
trigger: always_on
description: 数据持久化层规范（repositories 模式、加密、表结构）
---

# 数据持久化层规范

## 架构概述

位置：[backend/app/repositories.py](mdc:backend/app/repositories.py)

Repository 模式封装所有数据库访问，提供：
- 统一的读写接口
- 数据加密（凭据）
- 类型安全的输入输出
- 与业务逻辑解耦

## 连接管理

### 获取连接
```python
from backend.app.db import get_connection

with get_connection() as conn:
    rows = conn.execute("SELECT * FROM symbols").fetchall()
```

**注意**：
- 使用 `with` 上下文管理器确保连接关闭
- `get_connection()` 自动执行迁移和列补丁
- 参见 [db-duckdb](mdc:.cursor/rules/db-duckdb.mdc) 规则

## 凭据管理

### 保存凭据（加密）
```python
from backend.app.repositories import save_credentials

creds = {
    "LONGPORT_APP_KEY": "your_key",
    "LONGPORT_APP_SECRET": "your_secret",
    "LONGPORT_ACCESS_TOKEN": "your_token"
}

save_credentials(creds)
```

**实现细节**：
- 使用 Fernet 对称加密（配置在 `config.py`）
- 存储到 `settings` 表（key/value 结构）
- 键映射：`LONGPORT_APP_KEY` → `longport_app_key`

### 加载凭据（解密）
```python
from backend.app.repositories import load_credentials

creds = load_credentials()
# 返回：{"LONGPORT_APP_KEY": "...", "LONGPORT_APP_SECRET": "...", ...}
```

**错误处理**：
- 解密失败时返回空字符串
- 不抛出异常（优雅降级）

## 符号列表管理

### 保存符号列表
```python
from backend.app.repositories import save_symbols

symbols = ["AAPL.US", "TSLA.US", "700.HK"]
save_symbols(symbols)
```

**特性**：
- 自动去重、排序、大写转换
- 删除旧列表后批量插入
- 设置 `enabled=1`

### 加载符号列表
```python
from backend.app.repositories import load_symbols

symbols = load_symbols()
# 返回：["700.HK", "AAPL.US", "TSLA.US"]  # 已排序
```

**表结构**：
```sql
CREATE TABLE symbols (
    symbol TEXT PRIMARY KEY,
    enabled INTEGER DEFAULT 1
)
```

## K 线数据管理

### 存储 K 线
```python
from backend.app.repositories import store_candlesticks

# candles 是 Longbridge SDK 返回的对象列表
count = store_candlesticks(symbol="AAPL.US", candles=candles)
# 返回成功写入的记录数
```

**实现细节**：
- 使用 `INSERT OR REPLACE` 避免重复
- 自动转换时间戳（支持 datetime 或 Unix 时间戳）
- 安全浮点数转换（`_safe_float`）
- 显式 `commit()` 保证事务完整性

**表结构**：
```sql
CREATE TABLE ohlc (
    symbol TEXT,
    ts TIMESTAMP,
    open REAL,
    high REAL,
    low REAL,
    close REAL,
    volume REAL,
    turnover REAL,
    PRIMARY KEY (symbol, ts)
)
```

### 读取 K 线
```python
from backend.app.repositories import fetch_candlesticks

bars = fetch_candlesticks(symbol="AAPL.US", limit=100)
# 返回最近 100 条 K 线（倒序）
```

**返回格式**：
```python
[
    {
        "ts": "2024-10-03 10:00:00",
        "open": 150.0,
        "high": 152.0,
        "low": 149.5,
        "close": 151.0,
        "volume": 1000000,
        "turnover": 151000000
    }
]
```

### 从 Ticks 聚合分钟线
```python
from backend.app.repositories import fetch_bars_from_ticks

bars = fetch_bars_from_ticks(symbol="AAPL.US", limit=60)
# 返回最近 60 分钟的聚合 K 线
```

**用途**：
- 历史 K 线未同步时的回退方案
- 使用 `date_trunc('minute', ts)` 分组聚合
- 性能优化：仅扫描最近 3 天数据

## Tick 数据管理

### 存储 Tick
```python
from backend.app.repositories import store_tick_event

# quote_event 是 Longbridge QuoteContext 推送的对象
store_tick_event(symbol="AAPL.US", quote_event=quote_event)
```

**实现细节**：
- 提取 `timestamp`, `last_done`, `volume`, `turnover` 等字段
- 支持多种时间戳格式（datetime、Unix 时间戳、字符串）
- 使用 `ON CONFLICT DO UPDATE` 避免重复

**表结构**：
```sql
CREATE TABLE ticks (
    symbol TEXT,
    ts TIMESTAMP,
    sequence INTEGER,
    price REAL,
    volume REAL,
    turnover REAL,
    current_volume REAL,
    current_turnover REAL,
    PRIMARY KEY (symbol, ts)
)
```

### 读取 Tick
```python
from backend.app.repositories import fetch_ticks

ticks = fetch_ticks(symbol="AAPL.US", limit=100)
# 返回最近 100 条 tick 数据
```

**返回格式**：
```python
[
    {
        "ts": "2024-10-03 10:00:00.123",
        "sequence": 123456,
        "price": 151.25,
        "volume": 1000000,
        "turnover": 151250000,
        "current_volume": 1000,
        "current_turnover": 151250
    }
]
```

## 持仓数据管理

### 保存持仓
```python
from backend.app.repositories import save_positions

positions = [
    {"symbol": "AAPL.US", "qty": 100, "avg_price": 150.0},
    {"symbol": "TSLA.US", "qty": 50, "avg_price": 250.0}
]

save_positions(positions)
```

**特性**：
- 删除旧数据后批量插入（快照模式）
- 自动添加 `last_updated` 时间戳
- 自动大写转换与数据清洗

### 加载持仓
```python
from backend.app.repositories import load_positions

positions = load_positions()
# 返回所有持仓（按 symbol 排序）
```

**表结构**：
```sql
CREATE TABLE positions (
    symbol TEXT PRIMARY KEY,
    qty REAL,
    avg_price REAL,
    last_updated TIMESTAMP
)
```

## 最新价格查询

### 批量查询最新价
```python
from backend.app.repositories import fetch_latest_prices

prices = fetch_latest_prices(["AAPL.US", "TSLA.US"])
# 返回：
# {
#   "AAPL.US": {"ts": "...", "price": 151.0, "volume": 1000000, "source": "tick"},
#   "TSLA.US": {"ts": "...", "price": 250.0, "volume": None, "source": "ohlc"}
# }
```

**优先级**：
1. 先查 `ticks` 表（实时数据）
2. 未找到则查 `ohlc` 表（历史收盘价）

**用途**：
- 组合估值（[backend/app/services.py](mdc:backend/app/services.py)）
- 实时盈亏计算

## 监控配置管理

### 仓位监控配置

#### 保存配置
```python
from backend.app.repositories import save_position_monitoring_config

config = {
    "symbol": "AAPL.US",
    "monitoring_status": "active",
    "strategy_mode": "auto",
    "enabled_strategies": ["strategy_1", "strategy_2"],
    "max_position_ratio": 0.1,
    "stop_loss_ratio": 0.05,
    "take_profit_ratio": 0.1,
    "cooldown_minutes": 30,
    "notes": "趋势跟踪"
}

save_position_monitoring_config(config)
```

**特性**：
- `enabled_strategies` 自动 JSON 序列化
- 使用 `INSERT OR REPLACE` 支持更新
- 自动更新 `updated_at` 时间戳

#### 读取配置
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majiajue/longbridge](https://github.com/majiajue/longbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
