---
trigger: always_on
description: Contexto del proyecto para agentes de IA. Léeme **antes** de tocar código.
---

# CLAUDE.md — Transavic

Contexto del proyecto para agentes de IA. Léeme **antes** de tocar código.

> **📚 Para profundizar en cualquier área:** ver `docs/arquitectura/` (5 documentos temáticos verificados contra código). Empezar por [`docs/arquitectura/README.md`](./docs/arquitectura/README.md) que tiene un mapa "si vas a tocar X, lee Y".

---

## 1. Qué es este proyecto

**Sistema interno de gestión de pedidos** para una distribuidora avícola en Lima, Perú que opera dos marcas comerciales bajo el mismo dueño:

- **Transavic** — marca principal (pollo, gallinas, menudencia)
- **Avícola de Tony** — segunda marca (mismo flujo)

**Dueño / cliente final:** Antonio Resurrección.
**Productos:** pollo (entero, despresado, filetes), carnes (res, cerdo), huevos.
**Modelo:** venta al por mayor y menor a restaurantes, mayoristas y consumidores finales.
**Cobertura operativa:** 18 distritos de Lima Metropolitana.
**Volumen actual:** ~30 pedidos/día, 6 motorizados, 4 asesoras, 1 admin.

**No es** un e-commerce público ni un marketplace. Es un **ERP ligero interno** para la operación diaria. Los clientes finales no se loguean — los pedidos los crean las asesoras al recibirlos por WhatsApp.

---

## 2. Stack técnico

| Área | Tecnología |
|---|---|
| Framework | **Next.js 15** (App Router, Server Components + Server Actions) |
| Lenguaje | **TypeScript** (`strict: true`) |
| UI | **TailwindCSS v4** + `react-icons` (Feather) |
| Auth | **NextAuth v5 beta** + Credentials provider + `bcrypt` |
| Base de datos | **Neon Postgres** vía `@neondatabase/serverless` (HTTP, no pool) |
| Validación | **zod** (en cada API route) |
| Drag & drop | `@hello-pangea/dnd` (fork mantenido de react-beautiful-dnd) |
| Mapas | `@react-google-maps/api` + Google Maps Platform (Maps JS, Directions, Geocoding, Places) |
| Imágenes a JPEG | `html-to-image` (para compartir tickets por WhatsApp) |
| Offline | `localStorage` (NO IndexedDB) — ver `src/lib/offline-queue.ts` |
| Hosting | **Vercel** (deploy continuo desde main) |

**No usar ORM.** Las queries son SQL directo con tagged template literals de Neon (`sql\`SELECT ... \``). Hay queries dinámicas con `sql.query(query, params)` cuando hace falta.

**No usar PWA con background GPS.** iOS lo bloquea. Para el repartidor estamos planificando envolver `/dashboard/mi-ruta` con **Capacitor** (wrapper nativo) para tener GPS en background.

---

## 3. Comandos

```bash
npm run dev      # Desarrollo local en http://localhost:3000
npm run build    # Build de producción
npm run start    # Servir build
npm run lint     # ESLint (next/core-web-vitals + next/typescript)
npm run seed     # ./scripts/seed.mjs — crea tablas users + pedidos + seed inicial
```

**Migraciones:** scripts .mjs manuales en `/scripts/`. **No hay sistema automatizado.** Ejecutarlos uno a uno:

```bash
node scripts/migrate-products.mjs
node scripts/migrate-estados.mjs
node scripts/migrate-direccion-mapa.mjs
node scripts/migrate-entregado-por.mjs
node scripts/migrate-despacho-v2.mjs
node scripts/run-migration.mjs            # agrega asesor_id a clientes
# scripts/migration_add_asesor_to_clientes.sql — ejecutar manualmente en Neon
```

Cuando agregues una migración nueva, **crear nuevo archivo `migrate-<feature>.mjs`** siguiendo el patrón existente (con `CREATE EXTENSION IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`, etc.). No modificar migraciones existentes.

---

## 4. Variables de entorno

Definidas en `.env` (no comiteado). Las críticas:

| Variable | Para qué |
|---|---|
| `DATABASE_URL` | Conexión Neon (pooled) |
| `DATABASE_URL_UNPOOLED` | Conexión Neon directa (para migraciones largas) |
| `AUTH_SECRET` | Firma JWT de NextAuth |
| `AUTH_URL` | URL base para callbacks NextAuth |
| `NEXT_PUBLIC_MAPS_API_KEY` | Google Maps JS (cliente) |
| `Maps_SERVER_KEY` | Google Directions / Geocoding (server-side) — **ojo el naming inusual** (camelCase con M mayúscula y guión bajo, NO `MAPS_SERVER_KEY` ni `GOOGLE_MAPS_SERVER_KEY`) |
| `BASE_LATITUDE`, `BASE_LONGITUDE` | Fallback de ubicación del almacén; la fuente real es la tabla `settings.base_location` |

`ADMIN_USER`/`ADMIN_PASSWORD` están en `.env` pero **no se usan en código activo** (legacy del scaffolding inicial). La auth real lee de la tabla `users`.

---

## 5. Estructura del código (alto nivel)

```
src/
├── app/
│   ├── api/                      # Backend (Route Handlers)
│   │   ├── pedidos/              # CRUD de pedidos + transiciones
│   │   ├── despacho/             # Vista admin de despacho (kanban + asignación + ruta)
│   │   ├── repartidor/mi-ruta/   # Endpoint específico del repartidor
│   │   ├── clientes/, productos/, users/, analytics/, settings/, resumen-diario/
│   │   └── version/              # BUILD_ID para VersionChecker
│   ├── dashboard/                # UI con auth obligatoria
│   │   ├── (rutas por feature)/
│   │   └── layout.tsx            # Aplica DashboardLayout con sidebar
│   ├── login/, layout.tsx, page.tsx
├── components/                   # Componentes compartidos cross-feature
├── lib/
│   ├── types.ts                  # Pedido, Cliente, User, EstadoPedido, etc.
│   ├── data.ts                   # Queries reutilizables (fetchFilteredPedidos, fetchAsesores...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HugoHerreraCoach/transavic](https://github.com/HugoHerreraCoach/transavic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
