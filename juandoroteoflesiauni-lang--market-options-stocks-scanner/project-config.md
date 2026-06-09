---
trigger: always_on
description: Reglas de testing — asegurar que el código funciona antes de operar con dinero real
---


# 🧪 TESTING — TRADING TERMINAL

## FILOSOFÍA: CÓDIGO SIN TEST = CÓDIGO ROTO

En una terminal de trading, un bug puede significar pérdida de dinero.
**Cada función de lógica de negocio DEBE tener al menos un test.**

---

## 🐍 TESTS BACKEND (Python/Pytest)

### Estructura de tests:
```
tests/
├── unit/                    ← Tests de funciones individuales (rápidos)
│   ├── test_risk_service.py
│   ├── test_order_service.py
│   └── test_calculators.py
├── integration/             ← Tests de flujos completos (más lentos)
│   ├── test_order_flow.py
│   └── test_auth_flow.py
├── fixtures/                ← Datos de prueba compartidos
│   └── trading_fixtures.py
└── conftest.py              ← Configuración global de pytest
```

### Template de test unitario:
```python
# tests/unit/test_risk_service.py
import pytest
from decimal import Decimal
from unittest.mock import AsyncMock, patch

from app.services.risk_service import RiskService
from app.schemas.order_schema import OrderCreate
from app.core.exceptions import RiskViolationError, InsufficientFundsError

class TestRiskService:
    """Tests del servicio de gestión de riesgo."""
    
    @pytest.fixture
    def risk_service(self):
        return RiskService()
    
    @pytest.fixture
    def mock_portfolio(self):
        return {
            "available_usd": Decimal("5000"),
            "total_value": Decimal("10000")
        }
    
    # ========== HAPPY PATH ==========
    
    async def test_valid_order_passes_validation(self, risk_service, mock_portfolio):
        """Orden válida dentro de límites debe pasar."""
        order = OrderCreate(
            symbol="BTCUSDT",
            side="BUY",
            order_type="MARKET",
            quantity=Decimal("0.01")
        )
        current_price = Decimal("40000")
        
        # No debe lanzar excepción
        await risk_service.validate_order(order, mock_portfolio, current_price)
    
    # ========== CASOS BORDE ==========
    
    async def test_order_exceeding_max_size_raises_error(self, risk_service, mock_portfolio):
        """Orden > $10,000 debe ser rechazada."""
        order = OrderCreate(
            symbol="BTCUSDT",
            side="BUY",
            order_type="MARKET",
            quantity=Decimal("1.0")  # 1 BTC a $40k = $40,000
        )
        
        with pytest.raises(RiskViolationError) as exc_info:
            await risk_service.validate_order(order, mock_portfolio, Decimal("40000"))
        
        assert "excede límite" in str(exc_info.value)
    
    async def test_insufficient_funds_raises_error(self, risk_service):
        """Orden sin fondos suficientes debe ser rechazada."""
        poor_portfolio = {
            "available_usd": Decimal("100"),
            "total_value": Decimal("100")
        }
        order = OrderCreate(
            symbol="BTCUSDT",
            side="BUY",
            order_type="MARKET",
            quantity=Decimal("0.1")  # $4,000
        )
        
        with pytest.raises(InsufficientFundsError):
            await risk_service.validate_order(order, poor_portfolio, Decimal("40000"))
    
    async def test_negative_quantity_raises_error(self, risk_service, mock_portfolio):
        """Cantidad negativa debe ser rechazada por Pydantic."""
        with pytest.raises(ValueError):
            OrderCreate(
                symbol="BTCUSDT",
                side="BUY", 
                order_type="MARKET",
                quantity=Decimal("-1.0")
            )
```

---

## ⚡ TESTS FRONTEND (Vitest + Testing Library)

### Template de test de componente:
```typescript
// components/orders/__tests__/OrderForm.test.tsx

import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';
import OrderForm from '../OrderForm';
import * as orderService from '@/services/orderService';

// Mock del servicio
vi.mock('@/services/orderService');

describe('OrderForm', () => {
  const mockOnOrderPlaced = vi.fn();
  
  beforeEach(() => {
    vi.clearAllMocks();
  });
  
  // ========== RENDER ==========
  
  it('renders all required fields', () => {
    render(<OrderForm symbol="BTCUSDT" onOrderPlaced={mockOnOrderPlaced} />);
    
    expect(screen.getByLabelText(/cantidad/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /comprar/i })).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /vender/i })).toBeInTheDocument();
  });
  
  // ========== INTERACCIONES ==========
  
  it('submits buy order with correct data', async () => {
    const user = userEvent.setup();
    vi.mocked(orderService.placeOrder).mockResolvedValue({ orderId: '123' });
    
    render(<OrderForm symbol="BTCUSDT" onOrderPlaced={mockOnOrderPlaced} />);
    
    await user.type(screen.getByLabelText(/cantidad/i), '0.01');
    await user.click(screen.getByRole('button', { name: /comprar/i }));
    
    await waitFor(() => {
      expect(orderService.placeOrder).toHaveBeenCalledWith({
        symbol: 'BTCUSDT',
        side: 'BUY',
        quantity: 0.01,
        orderType: 'MARKET'
      });
    });
  });
  
  // ========== VALIDACIONES UI ==========
  
  it('shows error for negative quantity', async () => {
    const user = userEvent.setup();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
