---
trigger: always_on
description: - **V1** = `ValidApp_Web/` (producción local, puerto 5000) — **NUNCA TOCAR**
---

# AGENTS.md — Estado real del proyecto ValidApp

## Convenciones
- **V1** = `ValidApp_Web/` (producción local, puerto 5000) — **NUNCA TOCAR**
- **V2** = `ValidApp_Web_v2/` (desarrollo distribuidores, puerto 5001)
- **site-web/** = landing page Netlify (validaapp.netlify.app) — **NO TOCAR**

## Jerarquía de usuarios (capas)
```
SuperAdmin (VE TODO: capa directa + todas las capas distribuidor)
 │
 ├── [CAPA DIRECTA] Empresas directas (EMPRESA MASTER, TOTTYPETS)
 │   ├── Planes directos: Básico (200T), Intermedio (750T), Enterprise (2500T)
 │   ├── admin_empresa → cajero
 │   └── Validaciones descuentan del plan directo + plan global Pabilo
 │
 └── [CAPA DISTRIBUIDOR] Distribuidor A (COMSISTEC)
      ├── Pool de tokens: 2000T (comprado al SuperAdmin via plan_distribuidor)
      ├── Sus planes: "Plan Cliente 100T", "Plan Cliente 50T" (creados por él)
      ├── Empresa AA → admin_empresa → cajero
      └── Cada validación descuenta en CASCADA:
           1. Plan empresa (creado por distribuidor) -1T
           2. Pool del distribuidor -1T
           3. Plan global Pabilo -1T
```

## Regla clave de capas
- **SuperAdmin:** lo crea todo y lo ve todo
- **Distribuidor:** solo ve y opera su capa (sus planes, sus empresas, sus usuarios)
- **admin_empresa:** solo ve su empresa
- **cajero:** solo valida

## Lo que hace el distribuidor
- Compra planes al superadmin → recibe un pool de tokens con vigencia
- Crea sus propios planes (para sus empresas clientes)
- Crea empresas clientes y les asigna sus planes
- Crea usuarios (admin_empresa, cajero) para que sus clientes operen
- Monitorea KPIs, reportes y pagos de sus clientes
- Gestiona cuentas Pabilo de sus clientes

## Lo que NO hace el distribuidor
- ❌ Validar pagos (eso lo hacen sus clientes)
- ❌ Tener cuentas bancarias / Pabilo propias
- ❌ Consumir tokens directamente
- ❌ Ver historial de validaciones (nunca valida como distribuidor)
- ❌ session["empresa_id"] — NO aplica (no pertenece a ninguna empresa)

Si quiere validar → crea una empresa para sí mismo y usa un usuario de esa empresa.

## Base de datos Supabase
- URL: `https://wdtqxouqxdvsrkirctyp.supabase.co`
- Tabla `usuarios` columna `rol`: superadmin, admin_empresa, cajero, distribuidor
- `es_admin`: 1=superadmin, 2=admin_empresa, 0=resto
- NO existe tabla `roles` separada

### Empresas registradas
| ID | Nombre | RIF | Plan | Suscripción |
|----|--------|-----|------|-------------|
| 2 | EMPRESA MASTER | J-00000000-0 | 3 (Enterprise) | 2026-07-26 |
| 6 | TIENDA TOTTYPETS, C.A. | J-505800175 | 1 (Básico) | 2026-07-23 |

Ambas sin `distribuidor_id` ni `creado_por_usuario_id` — son empresas directas del superadmin.

### Usuarios
| ID | Username | Empresa | Rol |
|----|----------|---------|-----|
| 6 | admin | 2 (EMPRESA MASTER) | superadmin |
| 31 | supervisor | 2 | admin_empresa |
| 35 | cajademo | 2 | cajero |
| 38 | tvalida | 6 (TOTTYPETS) | cajero |
| 39 | leo | 6 | admin_empresa |
| 42 | apineda | 2 (EMPRESA MASTER) | distribuidor |

### Distribuidor actual
- **COMSISTEC, C.A.** (distribuidores.id=2, usuario_id=42, username: apineda)
- `usuario.empresa_id=2` — **BUG**: apunta a EMPRESA MASTER, debería ser 0/null
- Ninguna empresa tiene `distribuidor_id=2` → `_dist_empresa_ids()` retorna `[]`

### Planes del superadmin (para empresas directas)
| ID | Nombre | Tokens | Precio |
|----|--------|--------|--------|
| 1 | Plan Básico | 200 | $15 |
| 2 | Plan Intermedio | 750 | $45 |
| 3 | Plan Enterprise | 2500 | $120 |
| 4 | Plan TESTER | 10 | $0 |

## Problemas actuales en V2
1. **`crear_distribuidor_completo()` hardcodea `empresa_id: 2`** — el usuario distribuidor no debe tener `empresa_id` (debe ser 0 o null)
2. **`session["empresa_id"] = 2` en login** — el distribuidor NO debe tener `empresa_id` en sesión
3. **Distribuidor puede acceder a rutas de operación** (validar, APIs, historial) que no debería
4. **`agregar_empresa()` no guarda `distribuidor_id`** — empresas creadas por distribuidor no se vinculan
5. **Admin rutas usan `session["empresa_id"]`** en lugar de `_dist_empresa_ids()` para el distribuidor
6. **Faltan funcionalidades:** planes propios del distribuidor, asignación de planes a empresas

## Pendiente
Refactorizar módulo distribuidor en V2 desde cero con el modelo jerárquico correcto:
- `session["empresa_id"] = None` para distribuidor
- `usuario.empresa_id = 0` al crear distribuidor
- Bloquear rutas de operación al distribuidor
- Adaptar admin rutas para que usen `_dist_empresa_ids()`
- Sistema de planes propios del distribuidor
- Vincular `distribuidor_id` al crear empresa

## Modelo de consumo de tokens (cascada)
Por cada validación de una empresa que pertenece a un distribuidor:
1. **-1 al plan de la empresa** (plan creado por el distribuidor, ej: 100T → 99T)
2. **-1 al pool del distribuidor** (ej: 2000T → 1999T)
3. **-1 al plan global Pabilo** (igual que siempre)

Si falla alguno de los 3, la validación se rechaza (sin tokens en algún nivel).

---

# Reglas del Agente

## 1. Perfil y Rol
- **Rol:** Desarrollador Full-Stack especializado en integraciones fintech y automatización de conciliación bancaria.
- **Proyecto:** ValidaApp (Plataforma en tiempo real para validación bancaria de transacciones).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Panacripto/validapp-v2](https://github.com/Panacripto/validapp-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
