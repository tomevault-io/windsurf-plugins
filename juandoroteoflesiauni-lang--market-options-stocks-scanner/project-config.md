---
trigger: always_on
description: Reglas de componentes UI para la terminal de trading — gráficos, órdenes, portfolio
---


# 🖥️ UI COMPONENTS — TRADING TERMINAL

## DISEÑO DE LA TERMINAL DE TRADING

### Tema visual obligatorio:
```css
/* design-tokens.css — Variables globales */
:root {
  /* Colores base — Tema oscuro como Bloomberg/TradingView */
  --bg-primary: #0d1117;
  --bg-secondary: #161b22;
  --bg-panel: #1c2128;
  --bg-card: #21262d;
  
  /* Colores de trading */
  --color-buy: #00c851;        /* Verde — Compra */
  --color-sell: #ff4444;       /* Rojo — Venta */
  --color-neutral: #f0c040;    /* Amarillo — Neutro/Pendiente */
  
  /* Texto */
  --text-primary: #e6edf3;
  --text-secondary: #8b949e;
  --text-muted: #484f58;
  
  /* Bordes */
  --border-default: #30363d;
  --border-accent: #388bfd;
  
  /* Tipografía */
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace; /* Para precios */
  --font-ui: 'Inter', system-ui, sans-serif;
}
```

---

## 📊 COMPONENTES DE PRECIOS — Reglas Críticas

```typescript
// ✅ CORRECTO — Precio con color dinámico y fuente monoespaciada

interface PriceDisplayProps {
  price: number;
  previousPrice?: number;
  decimals?: number;
  showChange?: boolean;
}

const PriceDisplay: React.FC<PriceDisplayProps> = ({
  price,
  previousPrice,
  decimals = 2,
  showChange = false
}) => {
  const isUp = previousPrice !== undefined && price > previousPrice;
  const isDown = previousPrice !== undefined && price < previousPrice;
  const changeColor = isUp ? 'text-[#00c851]' : isDown ? 'text-[#ff4444]' : 'text-[#e6edf3]';
  
  return (
    <span 
      className={`font-mono font-semibold tabular-nums ${changeColor}`}
      aria-label={`Precio: ${price.toFixed(decimals)}`}
    >
      {price.toFixed(decimals)}
    </span>
  );
};

// REGLAS DE PRECIOS EN UI:
// 1. SIEMPRE usar font-mono para precios (alineación de dígitos)
// 2. SIEMPRE tabular-nums para evitar saltos visuales
// 3. Verde para subida, rojo para bajada
// 4. Decimales fijos según el instrumento (BTC=2, FOREX=5)
```

---

## 📋 FORMULARIO DE ÓRDENES

```typescript
// components/orders/OrderForm.tsx

interface OrderFormState {
  side: 'BUY' | 'SELL';
  orderType: 'MARKET' | 'LIMIT' | 'STOP_LIMIT';
  quantity: string;
  price: string;
  stopPrice: string;
}

const OrderForm: React.FC<{ symbol: string; onOrderPlaced: (id: string) => void }> = ({
  symbol,
  onOrderPlaced
}) => {
  const [state, setState] = useState<OrderFormState>({
    side: 'BUY',
    orderType: 'MARKET',
    quantity: '',
    price: '',
    stopPrice: ''
  });
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  // Validación del formulario
  const validateForm = (): string | null => {
    const qty = parseFloat(state.quantity);
    if (isNaN(qty) || qty <= 0) return 'Cantidad debe ser un número positivo';
    if (state.orderType === 'LIMIT' && !state.price) return 'Precio requerido para orden LIMIT';
    return null;
  };
  
  const handleSubmit = async () => {
    setError(null);
    const validationError = validateForm();
    if (validationError) { setError(validationError); return; }
    
    setIsLoading(true);
    try {
      const order = await orderService.placeOrder({
        symbol,
        side: state.side,
        orderType: state.orderType,
        quantity: state.quantity,
        price: state.price || undefined
      });
      onOrderPlaced(order.orderId);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error al enviar orden');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div className="bg-[#1c2128] border border-[#30363d] rounded-lg p-4">
      {/* Selector BUY/SELL */}
      <div className="grid grid-cols-2 gap-1 mb-4">
        <button
          onClick={() => setState(s => ({ ...s, side: 'BUY' }))}
          className={`py-2 rounded font-semibold transition-colors ${
            state.side === 'BUY' 
              ? 'bg-[#00c851] text-black' 
              : 'bg-[#21262d] text-[#8b949e] hover:bg-[#00c851]/20'
          }`}
        >
          COMPRAR
        </button>
        <button
          onClick={() => setState(s => ({ ...s, side: 'SELL' }))}
          className={`py-2 rounded font-semibold transition-colors ${
            state.side === 'SELL' 
              ? 'bg-[#ff4444] text-white' 
              : 'bg-[#21262d] text-[#8b949e] hover:bg-[#ff4444]/20'
          }`}
        >
          VENDER
        </button>
      </div>
      
      {/* Campo Cantidad */}
      <div className="mb-3">
        <label className="block text-[#8b949e] text-xs mb-1">Cantidad</label>
        <input
          type="number"
          value={state.quantity}
          onChange={e => setState(s => ({ ...s, quantity: e.target.value }))}
          placeholder="0.00"
          min="0"
          step="any"
          className="w-full bg-[#21262d] border border-[#30363d] text-[#e6edf3] 
                     font-mono rounded px-3 py-2 focus:border-[#388bfd] outline-none"
        />
      </div>
      
      {/* Error */}
      {error && (
        <div role="alert" className="text-[#ff4444] text-sm mb-3 p-2 bg-[#ff4444]/10 rounded">
          ⚠️ {error}
        </div>
      )}
      
      {/* Submit */}
      <button
        onClick={handleSubmit}
        disabled={isLoading}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
