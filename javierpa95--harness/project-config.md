---
trigger: always_on
description: **Proyecto:** [PROJECT_NAME] — [ONE_LINE_DESCRIPTION]
---

# AGENTS.md — Template de Proyecto (SDD Agent Harness)

**Proyecto:** [PROJECT_NAME] — [ONE_LINE_DESCRIPTION]
**Stack:** [STACK_TECH]
**Version:** 0.1.0
**Ultima actualizacion:** [DATE]

---

## Project Clarification — Primer Paso Obligatorio

> **Cuando se inicia un proyecto nuevo desde esta plantilla, el agente DEBE ejecutar este flujo antes de escribir codigo.**

### Paso 1: Identificar el proyecto

Pregunta al usuario y define:

| Pregunta | Ejemplo |
|----------|---------|
| **Nombre del proyecto** | `mi-tienda`, `taskflow`, `blog-dev` |
| **Descripcion en 1 linea** | "Tienda online de productos artesanales" |
| **Tipo de proyecto** | Web app, API, CLI, Mobile, Desktop, Full-stack |
| **Idioma de la UI** | Espanol, Ingles, Multi-idioma |

### Paso 2: Definir el stack

| Pregunta | Ejemplo |
|----------|---------|
| **Frontend** | Astro, Next.js, React, Vue, Svelte, None |
| **Backend** | PocketBase, Node/Express, Python/FastAPI, Supabase, None |
| **Database** | SQLite, PostgreSQL, MongoDB, None |
| **Deploy** | Coolify, Vercel, Railway, Docker, Manual |
| **Otros** | Auth provider, payment gateway, etc. |

### Paso 3: Decidir estructura de carpetas

Basandote en el stack, propone una estructura. Patrones comunes:

```
# Full-stack web
apps/
  web/                    Frontend
services/
  backend/                Backend + DB

# Monorepo multiple apps
apps/
  web/
  admin/
  api/
packages/
  shared/
  ui/

# Single app (simple)
src/
  components/
  pages/
  services/
  utils/
```

### Paso 4: Nombrar el agente arquitecto

El arquitecto principal se nombra como `[project]-architect`. Ejemplos:
- Tienda Marysol -> `store-architect`
- TaskFlow -> `taskflow-architect`
- MiBlog -> `blog-architect`

Renombra el archivo `.opencode/agents/project-architect.md` y actualiza su frontmatter.

### Paso 5: Configurar agentes relevantes

Elimina agentes que no apliquen al stack. Anade nuevos si es necesario.

| Agente template | Cuando usarlo |
|-----------------|---------------|
| `spec-writer` | Siempre — escribe feature specs |
| `frontend-developer` | Si hay frontend (web, mobile, desktop UI) |
| `backend-developer` | Si hay backend/API/database |
| `code-reviewer` | Siempre — revisa implementacion (2 ejes: Standards + Spec) |
| `gdpr-auditor` | Si manejas datos de usuarios |
| `release-manager` | Si necesitas versionado formal |

### Paso 6: Actualizar este archivo

Rellena los placeholders del header y las secciones siguientes con la info del proyecto.

### Paso 7: Crear CONTEXT.md

Crea `CONTEXT.md` en la raiz del proyecto con el glosario de dominio. Define los terminos clave que los agentes usaran.

---

## Flujo SDD + TDD — Specification-Driven Development

Este proyecto sigue el patron SDD. **Nada se implementa sin una spec. Nada se commitea sin review.**
Ademas, aplica **TDD (Test-Driven Development)** en backend y utils/shared.

### El Ciclo

```
1. ANALYZ    -> Architect analiza la peticion (o hace grilling si el plan es complejo)
2. SPEC      -> Spec Writer crea/actualiza la spec
3. IMPLEMENT -> Developers implementan (con TDD en backend/utils)
4. REVIEW    -> Code Reviewer verifica (2 ejes: Standards + Spec)
5. DECIDE    -> Architect: Pasa (commit) o itera?
```

### TDD — Donde aplica y donde no

| Capa | TDD | Por que |
|------|-----|---------|
| **Backend** (API, logica, DB, auth) | **SI** | Logica pura, facil de testear, alto valor |
| **Utils/Shared** (helpers, validators) | **SI** | Funciones puras, tests simples |
| **Frontend** (UI, componentes, paginas) | **NO** | Complejo de testear, menor ROI |

### TDD Cycle (Red -> Green -> Refactor)

```
1. RED     -> Escribe test que falla (comportamiento esperado)
2. GREEN   -> Escribe el minimo codigo para que pase
3. REFACTOR -> Mejora el codigo sin romper tests
4. REPITE  -> Siguiente comportamiento
```

### Reglas del Flujo

- **Spec primero**: No se escribe codigo sin una spec aprobada en `docs/features/`.
- **TDD en backend/utils**: Tests antes del codigo. Cada acceptance criteria de la spec se traduce en al menos un test.
- **Review obligatorio**: Todo cambio funcional pasa por code-reviewer (2 ejes). Solo se skippea en cambios triviales (texto, color, formateo).
- **Docs obligatorio**: Todo cambio funcional pasa por docs-auditor antes de commit. Si falta documentacion, se actualiza primero.
- **Paralelismo**: Frontend y backend se implementan en paralelo si ambos son necesarios.
- **Seguridad en paralelo**: Si hay datos sensibles, `code-reviewer` + `gdpr-auditor` corren simultaneamente.
- **Architect decide**: Solo el architect puede marcar una tarea como done o pedir iteracion.
- **Grilling**: Si el plan es complejo, el architect hace entrevista relajada antes del flujo SDD.

### Excepciones

| Caso | Flujo |
|------|-------|
| Cambio trivial (texto, color) | Analyze -> Implement -> Decide (skip spec + review) |
| Bug fix sin cambio de comportamiento | Analyze -> Implement -> Decide |
| Bug fix que cambia comportamiento | Flujo completo (spec obligatoria + tests) |
| Datos sensibles | Review + GDPR + Docs en paralelo |

---

## Commits — El Git Log es Documentacion

**El git log es la mejor documentacion de avance del proyecto.** Cada commit cuenta la historia de como evoluciono el codigo. Un historial limpio permite:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javierpa95/harness](https://github.com/javierpa95/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
