---
trigger: always_on
description: CRESIO es un sistema de gestión de activos TIC, inventario operativo, compras, recepción, custodias, mantenimientos, bodegas, stock y trazabilidad.
---

# AGENTS.md — Reglas de trabajo para CRESIO

## Contexto del proyecto

CRESIO es un sistema de gestión de activos TIC, inventario operativo, compras, recepción, custodias, mantenimientos, bodegas, stock y trazabilidad.

El proyecto está dividido en dos capas principales:

```text
Browser
  ↓
consumogestionactivosapi
BFF MVC / Thymeleaf / puerto 8081
  ↓ RestClient
gestionactivosapi
API Backend / puerto 8084
  ↓
PostgreSQL
```

`consumogestionactivosapi` es un BFF MVC/Thymeleaf.
No es el backend principal de negocio.
La persistencia, reglas profundas, entidades JPA, migraciones Flyway y lógica transaccional viven principalmente en `gestionactivosapi`.

## Stack

### BFF Web

* Java 17.
* Spring Boot.
* Spring MVC.
* Thymeleaf.
* Bootstrap 5.
* RestClient hacia API backend.
* Frontend Enterprise UI con clases `cui-*`.

### Backend API

* Java 17.
* Spring Boot.
* PostgreSQL.
* JPA.
* Flyway.
* Servicios de dominio/aplicación.
* Reglas transaccionales.

## Arquitectura funcional objetivo

CRESIO debe operar como plataforma ITAM empresarial, no como sistema de formularios aislados.

Regla UX principal:

```text
Datos visibles → Selección → Acción contextual → Drawer → Confirmación
```

No usar formularios permanentes si la acción puede nacer desde una fila.

## Reglas obligatorias generales

No modificar base de datos salvo aprobación explícita.

No modificar entidades, DTOs, servicios, controladores, endpoints o migraciones sin aprobación.

No crear endpoints nuevos sin aprobación.

No cambiar mappings existentes sin aprobación.

No cambiar nombres de inputs de formularios sin verificar DTOs BFF y backend.

No romper rutas antiguas.

No tocar Flutter salvo solicitud explícita.

No tocar API backend salvo que se indique explícitamente.

No mezclar cambios visuales con cambios de dominio.

No implementar múltiples fases en una sola respuesta.

No agregar funcionalidades especulativas.

No usar datos inventados para llenar vacíos del modelo.

## Flujo de trabajo obligatorio

Antes de modificar archivos:

1. Analizar código relacionado.
2. Identificar capa afectada:

   * BFF MVC/Thymeleaf.
   * API backend.
   * BD/Flyway.
   * UI estática.
3. Explicar el problema.
4. Proponer plan.
5. Listar archivos a crear/modificar.
6. Indicar riesgos.
7. Esperar aprobación.

Después de modificar:

1. Mostrar archivos creados/modificados.
2. Explicar cambios.
3. Ejecutar:

   * `.\mvnw.cmd compile`
   * `.\mvnw.cmd test`
   * `git diff --check`
4. Reportar resultado de validaciones.
5. Reportar riesgos.
6. No continuar con la siguiente fase sin aprobación.

## Reglas de Compras y Recepción

Regla central:

```text
Compra define lo solicitado.
Recepción convierte lo recibido en activo o stock.
```

Flujo objetivo:

```text
OrdenCompra
  ↓
OrdenCompraDetalle
  ↓
RecepcionLote
  ↓
Equipo / StockConsumibleBodega
  ↓
MovimientoInventario
  ↓
Custodia / Traslado / Mantenimiento / Baja
```

No debe existir recepción libre desconectada de una OC.

No usar pantallas independientes como solución principal:

```text
Recepción de activo
Recepción de consumible
```

La recepción debe realizarse desde:

```text
Gestionar OC → Línea de detalle → Recibir
```

Cada línea de OC debe mostrar:

* Descripción.
* Tipo: `ACTIVO` o `STOCK`.
* Cantidad solicitada.
* Cantidad recibida.
* Cantidad pendiente.
* Estado.
* Acción contextual `Recibir`.

Si el detalle es `ACTIVO`:

* Genera activos individuales.
* Requiere serial.
* Debe asociarse a OC, detalle y lote de recepción.
* Debe generar código CRESIO.
* Debe quedar vinculado a bodega.
* Debe guardar condición al recibir.
* Puede registrar datos técnicos opcionales.

Si el detalle es `STOCK`:

* No genera código individual.
* Incrementa stock en bodega.
* Debe asociarse a OC, detalle y lote de recepción.
* Debe generar movimiento de inventario.

## Estados de Orden de Compra

Estados válidos:

```text
BORRADOR
EMITIDA
RECEPCION_PARCIAL
RECIBIDA
CANCELADA
```

Reglas:

* `BORRADOR` permite edición.
* `EMITIDA` permite recepción.
* `RECEPCION_PARCIAL` permite recepción pendiente.
* `RECIBIDA` no permite nuevas recepciones.
* `CANCELADA` no permite operaciones.

Si en el sistema legacy existe `RECIBIDA_PARCIAL`, tratarlo como valor de compatibilidad temporal. No introducirlo en nuevas vistas como estado principal.

## Estados de OrdenCompraDetalle

Estados válidos:

```text
PENDIENTE
PARCIAL
COMPLETO
CANCELADO
```

Reglas:

* `cantidadRecibida = 0` → `PENDIENTE`.
* `0 < cantidadRecibida < cantidadSolicitada` → `PARCIAL`.
* `cantidadRecibida = cantidadSolicitada` → `COMPLETO`.
* No permitir `cantidadRecibida > cantidadSolicitada`.

## RecepcionLote

Todo lote debe tener:

* `uuid`.
* OC asociada.
* Detalle OC asociado.
* Cantidad recibida.
* Estado.
* Bodega destino.
* `recepcionadoPor`.
* `recepcionadoEn`.
* Observación opcional.

No usar `id_custodio_receptor` como responsable de recepción logística en nuevas fases. La recepción la ejecuta un usuario/custodio de bodega, pero la custodia del activo ocurre después.

## Reglas de bodegas

Una bodega debe tener custodio responsable.

El custodio responsable de bodega debe:

* Estar activo.
* Tener cargo/departamento asociado.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maycolmoreno/InvTICS](https://github.com/maycolmoreno/InvTICS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
