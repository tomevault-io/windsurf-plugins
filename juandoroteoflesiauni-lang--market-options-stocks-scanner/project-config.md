---
trigger: always_on
description: Reglas de manejo de errores y logging — prevenir fallos silenciosos en trading
---


# 🚨 MANEJO DE ERRORES Y LOGGING — TRADING TERMINAL

## FILOSOFÍA: FAIL LOUDLY, NOT SILENTLY

En trading, un error silencioso puede significar:
- Orden que se envió pero no se registró
- Pérdida calculada incorrectamente
- Precio desactualizado sin que el usuario lo sepa

**Cada error DEBE ser visible: al usuario, en los logs, o ambos.**

---

## 🐍 JERARQUÍA DE EXCEPCIONES (Python)

```python
# core/exceptions.py — Mapa completo de excepciones del dominio

class TradingError(Exception):
    """Base de todas las excepciones. Nunca usar directamente."""
    def __init__(self, message: str, code: str = "TRADING_ERROR"):
        self.message = message
        self.code = code
        super().__init__(message)

# ── Errores de autenticación ──────────────────────────
class UnauthorizedError(TradingError):
    """JWT inválido, expirado, o usuario sin permisos."""
    def __init__(self, message: str = "No autorizado"):
        super().__init__(message, "UNAUTHORIZED")

# ── Errores de validación ──────────────────────────────
class ValidationError(TradingError):
    """Input del usuario es inválido."""
    def __init__(self, message: str, field: str = ""):
        self.field = field
        super().__init__(message, "VALIDATION_ERROR")

class InvalidSymbolError(ValidationError):
    """El símbolo de trading no existe o no está disponible."""
    pass

# ── Errores financieros ────────────────────────────────
class InsufficientFundsError(TradingError):
    """No hay fondos suficientes para la operación."""
    def __init__(self, required: float, available: float):
        super().__init__(
            f"Fondos insuficientes: necesitas ${required:.2f}, tienes ${available:.2f}",
            "INSUFFICIENT_FUNDS"
        )
        self.required = required
        self.available = available

class RiskViolationError(TradingError):
    """La operación viola las reglas de gestión de riesgo."""
    def __init__(self, message: str, rule: str = ""):
        self.rule = rule
        super().__init__(message, "RISK_VIOLATION")

# ── Errores de órdenes ─────────────────────────────────
class OrderNotFoundError(TradingError):
    """La orden no existe o no pertenece al usuario."""
    pass

class OrderAlreadyCancelledError(TradingError):
    """La orden ya fue cancelada y no se puede modificar."""
    pass

class OrderExecutionError(TradingError):
    """Error al ejecutar la orden en el exchange."""
    pass

# ── Errores de exchange ────────────────────────────────
class ExchangeConnectionError(TradingError):
    """Error de conexión con el exchange."""
    pass

class ExchangeRateLimitError(TradingError):
    """Se alcanzó el límite de solicitudes del exchange."""
    pass
```

---

## 🔧 HANDLERS GLOBALES DE ERRORES (FastAPI)

```python
# main.py — Registrar handlers en el app de FastAPI

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(TradingError)
async def trading_error_handler(request: Request, exc: TradingError) -> JSONResponse:
    """Handler para todas las excepciones de negocio."""
    # Determinar HTTP status según el tipo de error
    status_map = {
        "UNAUTHORIZED":       401,
        "INSUFFICIENT_FUNDS": 402,
        "VALIDATION_ERROR":   422,
        "RISK_VIOLATION":     422,
        "TRADING_ERROR":      500,
    }
    status_code = status_map.get(exc.code, 500)
    
    logger.warning("trading.error",
                   code=exc.code,
                   message=exc.message,
                   path=str(request.url),
                   method=request.method)
    
    return JSONResponse(
        status_code=status_code,
        content={
            "error": {
                "code": exc.code,
                "message": exc.message,
                # NUNCA incluir stack trace en response al cliente
            }
        }
    )

@app.exception_handler(RequestValidationError)
async def validation_error_handler(request: Request, exc: RequestValidationError) -> JSONResponse:
    """Handler para errores de validación de Pydantic."""
    errors = []
    for error in exc.errors():
        errors.append({
            "field": ".".join(str(loc) for loc in error["loc"]),
            "message": error["msg"],
        })
    
    return JSONResponse(
        status_code=422,
        content={"error": {"code": "VALIDATION_ERROR", "fields": errors}}
    )

@app.exception_handler(Exception)
async def generic_error_handler(request: Request, exc: Exception) -> JSONResponse:
    """Handler de último recurso — captura errores inesperados."""
    logger.critical("unhandled.exception",
                    exc_type=type(exc).__name__,
                    exc_message=str(exc),
                    path=str(request.url),
                    exc_info=True)  # Incluye stack trace en los LOGS (no en response)
    
    return JSONResponse(
        status_code=500,
        content={"error": {"code": "INTERNAL_ERROR", "message": "Error interno del servidor"}}
    )
```

---

## 📊 SISTEMA DE LOGGING (structlog)

```python
# core/logger.py — Logger estructurado para producción

import logging
import structlog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
