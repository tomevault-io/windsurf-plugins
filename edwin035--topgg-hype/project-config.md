---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Idioma: comentarios, mensajes de error y texto de UI en español (ver CLAUDE.md global).

## Qué es esto

SPA de e-commerce (**TopGG / TopLevel Shop**): venta de pines/gift cards de gaming.
Stack: **React 18 + Vite + TypeScript + TailwindCSS + shadcn/ui**, data fetching con
`@tanstack/react-query` + `axios`/`fetch`, formularios con `react-hook-form` + `zod`.

Es solo el **frontend**. Consume el backend NestJS del repo hermano
`../hype-integration-2026` (working directory adicional), desplegado en
`https://hypes.up.railway.app/api`. Ver [Relación con el backend](#relación-con-el-backend).

## Comandos

```bash
npm run dev        # dev server en http://localhost:5173  (el script fuerza --port 5173)
npm run build      # build de producción (Vite + SWC; NO hace typecheck)
npm run build:dev  # build en modo development
npm run preview    # previsualiza el build
npm run lint       # ESLint (flat config, eslint.config.js)
npm test           # vitest run (una pasada)
npm run test:watch # vitest en watch
```

- **No hay script de typecheck** y `build` no verifica tipos (usa SWC). Para chequear tipos:
  `npx tsc --noEmit`.
- **Un solo test:** `npx vitest run src/ruta/al/archivo.test.ts` o por nombre
  `npx vitest run -t "nombre del test"`.
- Tests: patrón `src/**/*.{test,spec}.{ts,tsx}`, entorno `jsdom`, setup en
  `src/test/setup.ts`.
- Alias de imports: **`@` → `src/`** (definido en `vite.config.ts`, `vitest.config.ts`,
  `tsconfig.*.json` y `components.json`).

## Arquitectura

### Capa de red (dos módulos, un solo cliente HTTP)

Todo pasa por **`src/lib/api/http.ts`** — es el único cliente fetch:
- `apiRequest<T>()` con aliases `http()` y `authClient()` (mismo cuerpo, nombres legacy).
- Base URL: `import.meta.env.VITE_API_URL` (fallback `https://hypes.up.railway.app/api`).
- Token Bearer leído de `localStorage["toplevel_access_token"]` (`STORE_TOKEN_KEY`).
- Errores normalizados en la clase `ApiError` (status + data + mensaje en español).

Sobre ese cliente hay **dos familias de endpoints** que apuntan al mismo backend pero a
dominios distintos:

- **`src/lib/api/*`** → dominio propio del backend: `auth` (login / me / changePassword),
  `users`, `sales`. Barril: `src/lib/api/index.ts`.
- **`src/lib/providers/*`** → passthrough a las rutas del proveedor **Pin Hype**
  (`/pin-hype/...`): `catalog` (collections/products/stock), `redeem` (pre-redeem),
  `reversal`. Barril: `src/lib/providers/index.ts`. Inyecta `PIN_HYPE_DEFAULTS`
  (country `CO`, currency **`USD`**, language `es`) definidos en `http.ts`.

Los hooks de negocio viven en `src/hooks/providers/*` (`useCatalogSections`, `useBuyPin`,
`useStock`, `useReversal`, ...) y envuelven esas funciones de endpoints.

### Autenticación (OJO: dos contextos distintos, no confundir)

- **`src/components/Auth/AuthProvider.tsx`** → estado de **sesión/usuario**. Hidrata desde
  el token en localStorage: `getMe()` y luego `getUserById()`. Expone `useAuth()`
  (`user`, `token`, `initializing`, `setToken`, `logout`, `refresh`). Escucha el evento
  `storage` para sincronizar entre pestañas.
- **`src/contexts/AuthContext.tsx`** → estado del **modal de login** (abierto/cerrado y
  vista: login/register/recover/verify/reset). Expone `useAuthDialog()`. Emite el evento
  `app:close-modals` al abrir.
- **`src/RequireAuth.tsx`** protege rutas: si no hay `user`, dispara `openLogin()` y
  redirige a `/`.

Orden de providers en `src/App.tsx`:
`QueryClientProvider > TooltipProvider > AuthProvider > AuthDialogProvider`.

### Rutas (`src/App.tsx`)

- **Públicas:** `/` (Index), `/catalogo`, `/aliados`.
- **Protegidas (`RequireAuth`):** `/producto/:id`, `/checkout/:id`, `/factura/:id`,
  `/perfil`, `/cambiar-contrasena`, `/historial`, `/pago/binance/success`.

### Flujo de compra (pago con Binance Pay)

**Regla de oro:** el pin de Hype (`pre-redeem`) NO se canjea hasta que el pago está
verificado por webhook. Antes de pagar, la venta vive `PENDIENTE` sin pines.

**Dos orígenes de venta (`Sale.origen`):** `HYPE` (id de producto < 1.000.000 → se entrega con
`pre-redeem` contra el proveedor) y `PROPIO` (id >= 1.000.000 → se entrega asignando un
**código del inventario local**, sin tocar Hype). El backend ramifica por origen en
`purchase()` (precio de la BD, moneda USD, stock = códigos libres) y en `fulfill()` (asigna N
códigos con lock por producto — `OwnFulfillmentService`, a prueba de doble venta). Ver la
sección de productos propios más abajo.

1. `CheckoutPage` → `checkout()` (`src/lib/api/checkout.ts` → `POST /checkout`). El backend
   valida el precio real contra el catálogo de Hype (el cliente **no** envía precios),
   **pre-chequea stock**, convierte la moneda de venta a USDT (USD va **1:1**, sin tasa
   manual; COP usaría `BINANCE_USDT_COP_RATE`, camino legacy), crea la venta
   `PENDIENTE` y una **orden hosted-checkout de Binance**. Devuelve
   `{ sale, checkoutUrl, amountUsdt, usdtCopRate, ... }`. **No canja.**
2. El front guarda `tg_pending_binance_sale` en localStorage, muestra la aclaración de
   cobro en USDT y redirige a `checkoutUrl` (Binance).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Edwin035/topgg-hype](https://github.com/Edwin035/topgg-hype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
