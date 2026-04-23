---
trigger: always_on
description: Este documento proporciona contexto e información relevante para Claude Code al trabajar en el proyecto Amauta.
---

# CLAUDE.md

## Información del Proyecto Amauta

Este documento proporciona contexto e información relevante para Claude Code al trabajar en el proyecto Amauta.

## Descripción del Proyecto

Amauta es un sistema educativo para la gestión del aprendizaje.

## Visión y Filosofía ⭐

**IMPORTANTE**: Antes de desarrollar, entender la visión del proyecto.

- **Documento de Visión**: `README.md` - Filosofía, principios de diseño, valores
- **Propósito**: Educación como derecho social, acceso universal, offline-first
- **Nombre**: "Amauta" (quechua) = maestro/sabio al servicio de la comunidad

> _"No concebimos la educación como un producto, sino como un derecho social."_

---

## 🚦 Estado Actual y Próximos Pasos

### Fase Actual: Fase 5 - Comunidad y Colaboración 🚧 En Progreso

**Progreso**: Sprint 16 en progreso — 4/9 issues completados

#### Completado en Fase 5:

- ✅ **F5-001**: Refinar historias y criterios de aceptación de comunidad — planning
- ✅ **F5-002**: Matriz de dependencias UI/Backend para foros y comunidad
  - Modelos Prisma a crear: `ForoPost`, `ForoRespuesta`, `ReaccionForo`, `Notificacion`
  - Grafo de dependencias: F5-004 → F5-005 → F5-006 / F5-007 → F5-008 (F5-006 y F5-007 en paralelo)
  - Orden verificado: correcto; F5-009 puede ir en paralelo con F5-006 y F5-007
  - Documentado en `docs/project-management/roadmap.md` → Sección "Matriz de Dependencias UI/Backend — Fase 5"
- ✅ **F5-003**: Diseño funcional de flujos de foros, reportes y mensajes — planning
  - Ciclo de vida de ForoPost (PUBLICADO → CERRADO → ELIMINADO, soft delete, transiciones por rol)
  - Flujo de solución: solo PREGUNTA, una por post, toggle si cambia, notificación al autor
  - Moderación directa por EDUCADOR (su curso) y ADMIN_ESCUELA (toda la institución)
  - Sin modelo Reporte formal en Fase 5 — moderación directa es suficiente para comunidades pequeñas
  - Mensajes directos: fuera del alcance de Fase 5 (Fase 6+)
  - Documentado en `docs/project-management/fase-5-diseno-funcional-foros.md`
- ✅ **F5-004**: Prisma base de comunidad — modelos `ForoPost`, `ForoRespuesta`, `ReaccionForo`, `Notificacion`
  - 3 enums nuevos: `TipoForoPost`, `EstadoForoPost`, `TipoNotificacion`
  - 4 modelos nuevos con relaciones y índices
  - Migración `20260409000100_comunidad_foros_base`
  - Documentado en `docs/ai-context/database/schema.md`

#### Próximos pasos:

- 📋 **F5-005**: API de foros por curso: crear/listar posts y respuestas

**Documento guía**: `docs/project-management/roadmap.md` → Sección "Fase 5"

---

### Fase Anterior: Fase 4 - Módulo Escolar ✅ COMPLETADA

**Progreso**: Sprint 15 completado ✅ (3/3 completados) — Fase 4 completa

#### Completado en Fase 4:

- ✅ **F4-001**: Refinar historias y criterios de aceptación (Fase 4) — planning
- ✅ **F4-002**: Matriz de dependencias UI/Backend (Fase 4) — planning
- ✅ **F4-003**: Diseño funcional de flujos administrativos (roles) — planning
- ✅ **F4-004**: API Periodos Académicos + Escala de Calificación (Institución)
  - Modelos Prisma: `PeriodoAcademico`, `EscalaCalificacion` + migración
  - Módulo `instituciones` con service, controller y DTOs Zod
  - CRUD de períodos por institución (soft delete con `activo = false`)
  - Endpoint upsert de escala de calificación por institución (1:1)
  - 24 tests (98% statements, 100% funciones)
- ✅ **F4-005**: API Gestión de Grupos/Clases (CRUD + estados)
  - Relación con `PeriodoAcademico` y filtros por periodo/estado
  - Validación de institución y educador
  - Tests unitarios de controller y service
- ✅ **F4-006**: UI Gestión de Grupos/Clases (Listado + Form)
  - Endpoint backend `GET /instituciones/mi-institucion` para ADMIN_ESCUELA
  - Componentes `GruposList` (filtros estado/período) y `GrupoForm` (crear/editar)
  - Páginas: `/dashboard/grupos`, `/dashboard/grupos/nuevo`, `/dashboard/grupos/[id]/editar`
  - Proxies Next.js: `/api/grupos`, `/api/grupos/[id]`, `/api/mi-institucion`
  - "Grupos" en el sidebar (solo ADMIN_ESCUELA)
  - 19 tests frontend (2 suites, 100% pasando)
- ✅ **F4-010**: API listado de estudiantes y educadores por institución
  - Endpoints `GET /instituciones/:id/estudiantes` y `GET /instituciones/:id/educadores`
  - Filtros opcionales por búsqueda (`buscar`) y paginación (`page`, `limit`)
  - Validación de permisos (ADMIN_ESCUELA/SUPER_ADMIN)
- ✅ **F4-009**: UI Asignación de estudiantes y educadores a grupos
  - Páginas: `/dashboard/grupos/[id]/estudiantes` y `/dashboard/grupos/[id]/educadores`
  - Selector de usuarios por institución con búsqueda y paginación
  - Preview de asignaciones (agregados/duplicados/errores)
  - Gestión de asignaciones actuales con remoción y confirmación
- ✅ **F4-011**: API Registro diario de asistencias por grupo
  - Endpoints `GET /grupos/:grupoId/asistencias` y `PUT /grupos/:grupoId/asistencias`
  - Nómina activa por fecha con asistencia existente del día
  - Registro masivo por grupo/fecha con estados `PRESENTE`, `AUSENTE`, `TARDANZA` y `JUSTIFICADO`
  - Permisos para `ADMIN_ESCUELA` y educadores asignados al grupo
  - Edición solo el mismo día con observación obligatoria cuando cambia un registro existente
  - Tests unitarios de controller y service + documentación del contrato público

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/informaticadiaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
