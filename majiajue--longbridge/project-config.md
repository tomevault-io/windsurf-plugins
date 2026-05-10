---
trigger: always_on
description: 异常处理与错误规范（自定义异常、路由层映射、日志）
---

# 异常处理规范

## 自定义异常

位置：[backend/app/exceptions.py](mdc:backend/app/exceptions.py)

### LongbridgeDependencyMissing
- **继承**：`RuntimeError`
- **场景**：未安装 `longport` Python SDK
- **用法**：
  ```python
  try:
      from longport.openapi import QuoteContext, Config
  except ModuleNotFoundError as exc:
      raise LongbridgeDependencyMissing(
          "longport Python SDK not found. Please run `pip install longport`."
      ) from exc
  ```
- **HTTP 映射**：503 Service Unavailable
- **解决方案**：运行 `pip install longport`

### LongbridgeAPIError
- **继承**：`RuntimeError`
- **场景**：
  - Longbridge API 调用失败
  - 凭据未配置或无效
  - 网络连接问题
  - 权限不足
- **用法**：
  ```python
  if not self.credentials:
      raise LongbridgeAPIError("Trading credentials not configured")
  
  try:
      response = ctx.submit_order(...)
  except Exception as e:
      raise LongbridgeAPIError(f"Failed to place order: {e}")
  ```
- **HTTP 映射**：502 Bad Gateway
- **日志级别**：ERROR

## 路由层异常处理

### FastAPI 异常映射

在路由中统一捕获与转换：

```python
from fastapi import APIRouter, HTTPException
from ..exceptions import LongbridgeDependencyMissing, LongbridgeAPIError

router = APIRouter()

@router.get("/example")
async def example_endpoint():
    try:
        # 业务逻辑
        result = some_service_call()
        return result
    
    except LongbridgeDependencyMissing as exc:
        # 503：服务不可用
        raise HTTPException(status_code=503, detail=str(exc))
    
    except LongbridgeAPIError as exc:
        # 502：上游服务错误
        raise HTTPException(status_code=502, detail=str(exc))
    
    except ValueError as exc:
        # 400：参数校验失败
        raise HTTPException(status_code=400, detail=str(exc))
    
    except Exception as exc:
        # 500：未知错误
        logger.error(f"Unexpected error: {exc}")
        raise HTTPException(status_code=500, detail="Internal server error")
```

### 标准 HTTP 状态码

- **400 Bad Request**：
  - 参数校验失败（`ValueError`）
  - FastAPI Pydantic 校验失败
  - 示例：`raise HTTPException(400, detail="Invalid symbol format")`

- **404 Not Found**：
  - 资源不存在
  - 示例：`raise HTTPException(404, detail="Symbol not found")`

- **500 Internal Server Error**：
  - 未捕获的异常
  - 数据库错误
  - 示例：`raise HTTPException(500, detail="Database error")`

- **502 Bad Gateway**：
  - Longbridge API 调用失败（`LongbridgeAPIError`）
  - 外部服务错误

- **503 Service Unavailable**：
  - SDK 未安装（`LongbridgeDependencyMissing`）
  - 服务未启动

## 日志记录

### 日志级别

使用 Python `logging` 模块：

```python
import logging
logger = logging.getLogger(__name__)

# DEBUG：调试信息
logger.debug(f"Processing quote for {symbol}")

# INFO：常规信息
logger.info(f"Strategy {strategy_id} enabled")

# WARNING：警告（不影响运行）
logger.warning(f"High volatility detected for {symbol}")

# ERROR：错误（影响功能）
logger.error(f"Failed to load credentials: {e}")

# CRITICAL：严重错误（系统级）
logger.critical("Database connection lost")
```

### 日志位置

- **后端日志**：`logs/backend.log`
- **前端日志**：`logs/frontend.log`

### 日志格式

配置在 [backend/app/main.py](mdc:backend/app/main.py)：

```python
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/backend.log'),
        logging.StreamHandler()
    ]
)
```

## 错误处理最佳实践

### 1. Services 层
抛出具体异常，不处理 HTTP：

```python
# backend/app/services.py
def get_portfolio_overview():
    try:
        ctx = _trade_context()
        return ctx.account_balance()
    except ModuleNotFoundError as exc:
        raise LongbridgeDependencyMissing("SDK not installed") from exc
    except Exception as exc:
        raise LongbridgeAPIError(f"Failed to get portfolio: {exc}") from exc
    finally:
        ctx.close()
```

### 2. Repositories 层
处理数据库异常：

```python
# backend/app/repositories.py
def save_credentials(creds: Dict[str, str]) -> None:
    try:
        with get_connection() as conn:
            # 数据库操作
            conn.execute(...)
    except Exception as e:
        logger.error(f"Failed to save credentials: {e}")
        raise  # 重新抛出，由上层处理
```

### 3. Routers 层
转换为 HTTP 响应：

```python
# backend/app/routers/portfolio.py
@router.get("/overview")
async def get_overview():
    try:
        overview = get_portfolio_overview()
        return overview
    except LongbridgeDependencyMissing as exc:
        raise HTTPException(503, detail=str(exc))
    except LongbridgeAPIError as exc:
        raise HTTPException(502, detail=str(exc))
```

### 4. 策略引擎
优雅降级，不中断系统：

```python
# backend/app/strategy_engine.py
async def process_kline(self, symbol: str, bar: Dict):
    try:
        # 策略评估
        await self.evaluate_strategy(...)
    except Exception as e:
        logger.error(f"Error processing K-line for {symbol}: {e}")
        # 不重新抛出，避免中断行情流
```

### 5. 仓位监控
单个仓位失败不影响其他仓位：

```python
# backend/app/position_monitor.py
async def process_quote(self, symbol: str, quote_data: Dict):
    try:
        async with self._lock:
            # 处理单个仓位
            position = self.monitored_positions[symbol]
            await self.evaluate_strategies(position, quote_data)
    except Exception as e:
        logger.error(f"Error monitoring {symbol}: {e}")
        # 继续监控其他仓位
```

## 前端错误处理

### API 客户端
位置：[frontend/src/api/client.ts](mdc:frontend/src/api/client.ts)

```typescript
async function handleResponse<T>(response: Response): Promise<T> {
  if (!response.ok) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majiajue/longbridge](https://github.com/majiajue/longbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
