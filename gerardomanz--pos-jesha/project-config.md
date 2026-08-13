---
trigger: always_on
description: **JESHA POS** is a multi-tenant Point of Sale (POS) system for hardware stores. It manages sales, inventory, purchases, billing, customer accounts, and multi-branch operations across multiple businesses (empresas) via tenant isolation.
---

# AGENTS.md — JESHA POS

## Project Overview

**JESHA POS** is a multi-tenant Point of Sale (POS) system for hardware stores. It manages sales, inventory, purchases, billing, customer accounts, and multi-branch operations across multiple businesses (empresas) via tenant isolation.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Vanilla HTML/CSS/JS (no framework) |
| Backend | Node.js + Express |
| Database | PostgreSQL via Prisma ORM |
| Authentication | JWT tokens |
| Image Storage | Cloudinary |
| Billing | Facturapi (CFDI 4.0) |

## Project Structure

```
Ferreteria JESHA/
├── index.html, punto-venta.html, productos.html, ...   # Frontend pages
├── config.js                                              # Central API config + IVA rate
├── sidebar.js, sidebar.html                               # Global navigation component
├── dashboard.js, dashboard.css                           # Main dashboard
├── jesha-pos-backend/
│   ├── src/
│   │   ├── app.js                                        # Express routes (public + protected) + /health + /health/db
│   │   ├── server.js                                      # Server entry point
│   │   ├── helpers/
│   │   │   ├── getEmpresaId.js                           # Tenant extraction from JWT (all creates)
│   │   │   └── unidades.helper.js                       # Catálogo de unidades + normalización + inferencia
│   │   ├── middlewares/auth.middleware.js                # JWT auth + role guards + sucursal access
│   │   ├── lib/prisma.js, cloudinary.js                 # Prisma + Cloudinary clients
│   │   ├── utils/
│   │   │   └── roles.js                                  # Role hierarchy (JERARQUIA_ROLES)
│   │   └── modules/
│   │       ├── auth/                                      # Login (public)
│   │       ├── ventas/                                   # Sales + tickets
│   │       ├── productos/                                # Products + CSV import
│   │       ├── inventario/                              # Branch inventory
│   │       ├── clientes/                                 # Customer management
│   │       ├── turnos-caja/                             # Cash register shifts
│   │       ├── bitacora/                                # Customer accounts/ledger
│   │       ├── cotizaciones/                            # Quotes
│   │       ├── pedidos/                                 # Orders
│   │       ├── compras/                                 # Purchase orders
│   │       ├── devoluciones/                            # Returns
│   │       ├── facturacion/                             # Facturapi (public)
│   │       ├── facturas/                                # Invoice records
│   │       ├── impresion/                               # PrintJob management + snapshot builders
│   │       └── sucursal/                                # Branch helper + GET endpoint (CRUD parcial)
│   └── prisma/
│       └── schema.prisma                                # Full database schema
```

## Git Workflow — Cómo subir cambios a `main` (procedimiento explícito)

### Modelo de trabajo

- **Default: trabajo directo sobre `main`.** Los cambios de rutina (correcciones, ajustes de lógica, fixes puntuales) se commitean en `main` local y se empujan a `origin/main`. No se crean ramas para esto.
- **Excepción deliberada: rama + PR.** Solo para features grandes y aislables que el usuario pida explícitamente (ej. PR2 pago mixto ya integrado; PR3/PR4 de granel/pausadas/hover pendientes). Tras el merge, limpiar la rama local y remota como se hizo en PR2.
- **Regla de oro ante errores: NUNCA crear una rama para escapar de un problema.** Si algo sale mal, se resuelve sobre `main` con `stash`, `rebase` o `revert`. La proliferación de ramas divergentes es exactamente lo que se quiere evitar.
- **Producción es en vivo.** Cada push a `origin/main` despliega automáticamente el backend en Render. El frontend NO se despliega solo: requiere build manual (`build-frontend.ps1` en Windows, `build-frontend.sh` en Linux/macOS) + `npx wrangler deploy`. En push directo no hay gate de revisión, así que **probar en Brave contra el entorno local antes de empujar es obligatorio.**

### 1. Pre-flight (antes de tocar cualquier archivo)

```powershell
git switch main
git fetch origin
git status --short --branch
```

Esperado: `## main...origin/main` sin `ahead`/`behind`.
- Si dice `behind N`: `git pull --ff-only origin main` y reconfirmar.
- Si dice `ahead` inesperado o `diverged`: **PARAR** (ver tabla de recuperación).

### 2. Verificar ANTES de commitear

1. `node --check <archivo>` en **cada** `.js` tocado. Si falla, no continuar.
2. Probar en Brave contra local (`npm run dev`). Confirmar el comportamiento real, no asumirlo.
3. Revisar exactamente qué cambió:
   ```powershell
   git --no-pager diff
   git status --short
   ```
   `status` debe mostrar SOLO los archivos esperados. Si aparece algo más, **PARAR** y revisar antes de hacer stage.

### 2.1. Protocolo de pruebas locales (entrada al sistema + endpoints)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GerardoManz/POS-JESHA](https://github.com/GerardoManz/POS-JESHA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
