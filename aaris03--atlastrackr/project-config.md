---
trigger: always_on
description: App para repartidores: registra jornadas de trabajo, ingresos, gastos, vehículos y mantenimientos por kilometraje.
---

# AtlasTrackr — Contexto para Claude Code

## Qué es este proyecto
App para repartidores: registra jornadas de trabajo, ingresos, gastos, vehículos y mantenimientos por kilometraje.

---

## Stack

### Frontend (frontend/)
- React 19 + Vite 6
- React Router DOM 7 (SPA, sin SSR)
- Tailwind CSS 4
- Axios 1.x con interceptor de token en `src/api/axios.js`
- Estado global con Context API (`src/context/AuthContext.jsx`)

### Backend (backend/)
- Node.js 24 + Express 4
- Prisma 6 + PostgreSQL 18
- JWT (jsonwebtoken 9) — tokens de 7 días
- bcrypt 5 (10 rounds)
- Zod 3 para validación de schemas en todos los endpoints
- Jest + Supertest para tests de integración

---

## Estructura clave

```
AtlasTrackr/
├── frontend/src/
│   ├── api/axios.js              ← instancia axios con interceptor JWT
│   ├── context/AuthContext.jsx   ← estado global: user + vehículos
│   ├── components/               ← modales y Layout
│   ├── pages/                    ← Login, Register, Dashboard, Sessions, Expenses, Vehicles, Maintenance, Profile
│   └── utils/
│       ├── validators.js         ← validaciones de formularios
│       └── formatters.js         ← formatDate, formatCurrency
└── backend/
    ├── app.js                    ← Express + rutas + middlewares
    ├── server.js                 ← app.listen()
    ├── controllers/              ← lógica de negocio
    ├── middleware/auth.js        ← verifyToken (protege rutas privadas)
    ├── models/prisma.js          ← instancia PrismaClient (singleton)
    ├── routes/                   ← definición de rutas por módulo
    ├── utils/                    ← schemas Zod por módulo
    ├── test/                     ← Jest + Supertest (177 tests, todos pasando)
    └── prisma/
        ├── schema.prisma
        └── migrations/
```

---

## Convenciones del backend

### Estructura de un controlador
```js
// Importa desde models/prisma.js (nunca instanciar PrismaClient directo)
import prisma from '../models/prisma.js'

// Valida con Zod antes de tocar la DB
const parsed = schema.safeParse(req.body)
if (!parsed.success) return res.status(400).json({ error: parsed.error.issues })

// Respuestas: 200 OK, 201 Created, 400 bad input, 401 unauth, 403 forbidden, 404 not found, 409 conflict
```

### Rutas protegidas
Todas las rutas excepto `/api/auth/*` usan el middleware `verifyToken` de `middleware/auth.js`. El usuario autenticado queda en `req.user`.

### Soft delete vs Hard delete
- **Soft delete**: vehicles, maintenance_types, maintenance_records → setear `deleted_at = NOW()`, `is_active = false`
- **Hard delete**: expenses → DELETE directo. Al eliminar un expense → soft delete en cascade de sus `maintenance_records`

### Transacciones Prisma
Usar `prisma.$transaction()` en:
- Registro de usuario (crea 14 tipos de mantenimiento por defecto)
- Finalizar jornada
- Crear expense de tipo `maintenance` (crea `maintenance_record` automáticamente)

### Zona horaria
El frontend genera "hoy" con `getLocalDateString()` (`frontend/src/utils/formatters.js`), basado en hora local del navegador, no UTC. El backend valida fechas "no futuras" con tolerancia de ±1 día (`getDateStringWithOffset()` en `backend/utils/dateHelpers.js`) para absorber el desfase entre la fecha local del usuario (Chile, UTC-4) y la fecha UTC del servidor. `formatDate()` ya no usa el hack `setUTCHours(12)`.

---

## Convenciones del frontend

### Llamadas al backend
Siempre usar la instancia de `src/api/axios.js`, nunca `fetch` ni una instancia nueva de axios. El interceptor agrega el JWT automáticamente.

### Estado global
`AuthContext` maneja usuario autenticado y lista de vehículos. Consumir con `useContext(AuthContext)`. No duplicar este estado en componentes locales.

### Formularios
Validar con las funciones de `utils/validators.js` antes de llamar al backend.

### Modales
Pattern estándar: `show` (boolean) + `onClose` (función) como props. El estado de apertura vive en el padre (lifting state up).

---

## Modelos de datos clave

### work_sessions — estados
- `started` → jornada activa
- `finished` → jornada finalizada normalmente
- `voided` → jornada anulada (no cuenta en métricas)

### Cálculos de negocio
```
base_earnings  = orders × order_value
total_earnings = base_earnings + tips
km_work        = SUM(km_end - km_start) de jornadas finished con reserve_km = true
km_total       = km_actual - km_base
km_personal    = km_total - km_work
net_income     = total_earnings + base_salary - SUM(expenses.cost)
```

### vehicle_km_adjustments — reglas
- Diferencia ≤ 300 km → ajuste normal
- Diferencia > 300 km → requiere `force: true` (confirmación explícita del usuario)

---

## Comandos útiles

```bash
# Frontend
cd frontend && npm run dev         # Vite en localhost:5173

# Backend
cd backend && npm run dev          # nodemon en localhost:3000
cd backend && npm test             # Jest (177 tests de integración)

# Base de datos
cd backend && npx prisma studio    # GUI para explorar la DB
cd backend && npx prisma migrate dev --name <nombre>  # nueva migración
cd backend && npx prisma db seed   # seed inicial
```

---

## Estado actual del MVP

### Pendiente prioritario

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aaris03/AtlasTrackr](https://github.com/Aaris03/AtlasTrackr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
