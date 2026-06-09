---
trigger: always_on
description: Reglas para integración con exchanges — Binance, MT5 y adaptadores genéricos
---


# 🔗 INTEGRACIÓN CON EXCHANGES — TRADING TERMINAL

## PRINCIPIO: ADAPTADOR UNIVERSAL

El sistema NUNCA debe depender directamente de la API de un exchange específico.
Todos los exchanges se acceden a través de un adaptador que implementa la misma interfaz.
Esto permite cambiar de Binance a MT5 sin tocar el resto del código.

---

## 🏗️ INTERFAZ BASE DEL EXCHANGE

```python
# adapters/base_exchange.py

from abc import ABC, abstractmethod
from decimal import Decimal
from typing import AsyncIterator
from dataclasses import dataclass
from datetime import datetime

@dataclass
class OrderResult:
    order_id: str
    symbol: str
    side: str
    order_type: str
    quantity: Decimal
    fill_price: Decimal
    status: str
    timestamp: datetime
    exchange: str

@dataclass
class Ticker:
    symbol: str
    price: Decimal
    bid: Decimal
    ask: Decimal
    volume_24h: Decimal
    change_24h_pct: Decimal
    timestamp: datetime

@dataclass
class Balance:
    asset: str
    free: Decimal
    locked: Decimal
    
    @property
    def total(self) -> Decimal:
        return self.free + self.locked

class BaseExchangeAdapter(ABC):
    """
    Interfaz que TODOS los exchanges deben implementar.
    
    Si agregas un exchange nuevo, implementa esta clase.
    No modifiques el código que llama a esta interfaz.
    """
    
    @abstractmethod
    async def get_ticker(self, symbol: str) -> Ticker:
        """Obtener precio actual y datos de mercado."""
        ...
    
    @abstractmethod
    async def get_balances(self) -> list[Balance]:
        """Obtener todos los balances de la cuenta."""
        ...
    
    @abstractmethod
    async def place_order(
        self,
        symbol: str,
        side: str,          # "BUY" o "SELL"
        order_type: str,    # "MARKET" o "LIMIT"
        quantity: Decimal,
        price: Decimal | None = None
    ) -> OrderResult:
        """Enviar orden al exchange."""
        ...
    
    @abstractmethod
    async def cancel_order(self, symbol: str, order_id: str) -> bool:
        """Cancelar orden pendiente."""
        ...
    
    @abstractmethod
    async def get_order_status(self, symbol: str, order_id: str) -> OrderResult:
        """Consultar estado de una orden."""
        ...
    
    @abstractmethod
    async def stream_prices(self, symbol: str) -> AsyncIterator[Ticker]:
        """Stream de precios en tiempo real."""
        ...
```

---

## 🟡 ADAPTADOR BINANCE

```python
# adapters/binance_adapter.py

from binance import AsyncClient
from binance.exceptions import BinanceAPIException
from decimal import Decimal
from typing import AsyncIterator
import asyncio

from adapters.base_exchange import BaseExchangeAdapter, OrderResult, Ticker, Balance
from core.config import settings
from core.exceptions import ExchangeConnectionError, OrderExecutionError
from core.logger import logger

class BinanceAdapter(BaseExchangeAdapter):
    """Adaptador para Binance Spot Trading."""
    
    def __init__(self):
        self._client: AsyncClient | None = None
    
    async def _get_client(self) -> AsyncClient:
        """Lazy initialization del cliente."""
        if self._client is None:
            try:
                self._client = await AsyncClient.create(
                    api_key=settings.BINANCE_API_KEY,
                    api_secret=settings.BINANCE_API_SECRET,
                    testnet=settings.ENVIRONMENT != "production"  # Testnet en dev!
                )
            except Exception as e:
                raise ExchangeConnectionError(f"No se pudo conectar a Binance: {e}")
        return self._client
    
    async def get_ticker(self, symbol: str) -> Ticker:
        client = await self._get_client()
        try:
            data = await client.get_ticker(symbol=symbol)
            return Ticker(
                symbol=symbol,
                price=Decimal(data['lastPrice']),
                bid=Decimal(data['bidPrice']),
                ask=Decimal(data['askPrice']),
                volume_24h=Decimal(data['volume']),
                change_24h_pct=Decimal(data['priceChangePercent']),
                timestamp=datetime.fromtimestamp(data['closeTime'] / 1000)
            )
        except BinanceAPIException as e:
            logger.error("binance.ticker.error", symbol=symbol, error=str(e))
            raise ExchangeConnectionError(f"Error obteniendo precio de {symbol}: {e}")
    
    async def get_balances(self) -> list[Balance]:
        client = await self._get_client()
        try:
            account = await client.get_account()
            return [
                Balance(
                    asset=b['asset'],
                    free=Decimal(b['free']),
                    locked=Decimal(b['locked'])
                )
                for b in account['balances']
                if Decimal(b['free']) > 0 or Decimal(b['locked']) > 0
            ]
        except BinanceAPIException as e:
            raise ExchangeConnectionError(f"Error obteniendo balances: {e}")
    
    async def place_order(
        self,
        symbol: str,
        side: str,
        order_type: str,
        quantity: Decimal,
        price: Decimal | None = None
    ) -> OrderResult:
        client = await self._get_client()
        try:
            params = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
