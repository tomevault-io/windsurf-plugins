---
trigger: always_on
description: Reglas para manejo de datos en tiempo real, WebSockets y feeds de mercado
---


# ⚡ DATOS EN TIEMPO REAL — TRADING TERMINAL

## PRINCIPIOS DE TRADING EN TIEMPO REAL

Los datos de mercado llegan miles de veces por segundo.
Un error en este módulo = datos incorrectos = decisiones de trading equivocadas = pérdidas.

---

## 🔌 ARQUITECTURA WEBSOCKET BACKEND

```python
# backend/app/websockets/market_feed.py

from fastapi import WebSocket, WebSocketDisconnect
from typing import Dict, Set
import asyncio
import json

class MarketFeedManager:
    """
    Gestiona todas las conexiones WebSocket de clientes.
    Patrón: PubSub — un stream de exchange, N clientes.
    """
    
    def __init__(self):
        # symbol -> set de websockets conectados
        self._subscribers: Dict[str, Set[WebSocket]] = {}
        self._lock = asyncio.Lock()
    
    async def subscribe(self, websocket: WebSocket, symbol: str) -> None:
        """Suscribir cliente a un símbolo."""
        async with self._lock:
            if symbol not in self._subscribers:
                self._subscribers[symbol] = set()
                # Iniciar feed del exchange si es el primero
                asyncio.create_task(self._start_exchange_feed(symbol))
            self._subscribers[symbol].add(websocket)
    
    async def unsubscribe(self, websocket: WebSocket, symbol: str) -> None:
        """Desuscribir cliente — SIEMPRE llamar en disconnect."""
        async with self._lock:
            if symbol in self._subscribers:
                self._subscribers[symbol].discard(websocket)
                if not self._subscribers[symbol]:
                    # Limpiar stream si no hay más clientes
                    del self._subscribers[symbol]
    
    async def broadcast(self, symbol: str, data: dict) -> None:
        """Enviar datos a todos los clientes suscritos."""
        if symbol not in self._subscribers:
            return
        
        dead_connections = set()
        message = json.dumps(data)
        
        for websocket in self._subscribers[symbol].copy():
            try:
                await websocket.send_text(message)
            except Exception:
                dead_connections.add(websocket)
        
        # Limpiar conexiones muertas
        for ws in dead_connections:
            await self.unsubscribe(ws, symbol)
    
    async def _start_exchange_feed(self, symbol: str) -> None:
        """Conectar al exchange y retransmitir datos."""
        try:
            async for tick in exchange.get_price_stream(symbol):
                if symbol not in self._subscribers:
                    break  # No hay más suscriptores
                await self.broadcast(symbol, {
                    "type": "tick",
                    "symbol": symbol,
                    "price": str(tick.price),
                    "volume": str(tick.volume),
                    "timestamp": tick.timestamp.isoformat()
                })
        except Exception as e:
            logger.error(f"Exchange feed error for {symbol}: {e}")

feed_manager = MarketFeedManager()

# Endpoint WebSocket
@router.websocket("/ws/market/{symbol}")
async def market_websocket(websocket: WebSocket, symbol: str):
    await websocket.accept()
    await feed_manager.subscribe(websocket, symbol)
    try:
        while True:
            # Mantener conexión viva con heartbeat
            await websocket.receive_text()
    except WebSocketDisconnect:
        await feed_manager.unsubscribe(websocket, symbol)
```

---

## 🔌 ARQUITECTURA WEBSOCKET FRONTEND

```typescript
// hooks/useMarketFeed.ts

import { useEffect, useRef, useCallback } from 'react';
import { useMarketStore } from '@/store/marketStore';

interface MarketFeedOptions {
  symbol: string;
  onError?: (error: Event) => void;
}

export const useMarketFeed = ({ symbol, onError }: MarketFeedOptions) => {
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<ReturnType<typeof setTimeout>>();
  const updateTick = useMarketStore(s => s.updateTick);
  
  const connect = useCallback(() => {
    // Limpiar conexión anterior
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.close();
    }
    
    const ws = new WebSocket(`${import.meta.env.VITE_WS_URL}/market/${symbol}`);
    wsRef.current = ws;
    
    ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        if (data.type === 'tick') {
          updateTick(data.symbol, {
            price: parseFloat(data.price),
            volume: parseFloat(data.volume),
            timestamp: new Date(data.timestamp)
          });
        }
      } catch (e) {
        console.error('Error parsing market data:', e);
      }
    };
    
    ws.onerror = (error) => {
      console.error(`WS Error for ${symbol}:`, error);
      onError?.(error);
    };
    
    ws.onclose = (event) => {
      if (!event.wasClean) {
        // Reconexión automática con backoff exponencial
        reconnectTimeoutRef.current = setTimeout(connect, 3000);
      }
    };
  }, [symbol, updateTick, onError]);
  
  useEffect(() => {
    connect();
    
    // CRÍTICO: cleanup en unmount para evitar memory leaks
    return () => {
      clearTimeout(reconnectTimeoutRef.current);
      wsRef.current?.close(1000, 'Component unmounted');
      wsRef.current = null;
    };
  }, [connect]);
};

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
