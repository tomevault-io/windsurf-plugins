---
trigger: always_on
description: > **Última actualización:** 2026-02-15
---

# AGENTS.md — Sistema de Agentes para Desarrollo Móvil

> **Versión:** 2.0.0
> **Última actualización:** 2026-02-15
> **Propósito:** Definir roles, contratos, inputs/outputs y hand-offs para desarrollo de features móviles con agentes de IA.
> **Regla:** Si hay conflicto entre este archivo y cualquier otro, AGENTS.md prevalece.

---

## 1. Visión General del Sistema

Este sistema define un workflow repetible para construir features móviles (Android + iOS) desde un ticket hasta un PR listo para merge, asistido por agentes de IA.

### Principios

- **Contratos por etapa:** Cada agente produce artefactos con formato estándar
- **Gates explícitos:** No se avanza sin cumplir criterios de calidad
- **Herramienta-agnóstico:** Los roles no cambian si cambias Claude Code por Cursor, Copilot u otra herramienta
- **Spec Atómica:** Cada especificación debe ser autocontenida y ejecutable en un solo ciclo por el agente
- **Seguridad integrada:** OWASP MASVS se valida en cada etapa, no al final

### Estructura del Repositorio

```
AGENTS.md                          ← Este archivo (source of truth)
CLAUDE.md                          ← Contexto del proyecto para el agente
.claude/
  skills/
    owasp-mobile/SKILL.md          ← Revisión de seguridad OWASP MASVS
    api-consume/SKILL.md           ← Consumo seguro de APIs
    arch-check/SKILL.md            ← Guardia de Clean Architecture
    figma-to-ui/SKILL.md           ← Diseño a código nativo
  commands/
    spec.md                        ← /spec - Genera spec atómica
    api.md                         ← /api - Consume endpoint
    security.md                    ← /security - Auditoría OWASP
    ui.md                          ← /ui - Diseño a código
    arch.md                        ← /arch - Validar arquitectura
    pr.md                          ← /pr - Preparar PR
docs/
  features/
    FEAT-XXX/                      ← Una carpeta por feature
      spec.md
      ui_contract.md
      arch.md
      tasks.md
      qa_plan.md
      risks.md
      pr.md
  decisions.md                     ← ADRs (Architecture Decision Records)
rules/
  engineering.md                   ← Convenciones generales
  mobile_android.md                ← Reglas específicas Android
  mobile_ios.md                    ← Reglas específicas iOS
  security_privacy.md              ← Seguridad y privacidad
```

---

## 2. Flujo General

```
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 0: Bootstrap (1 vez por proyecto)                    │
│  → Configurar CLAUDE.md + AGENTS.md + /skills + /rules     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 1: Spec Agent                                        │
│  Ticket → spec.md + risks.md + tasks.md + qa_plan.md        │
│  Gate: ACs testables + 4 estados UI definidos               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 2: Design Translator                                 │
│  Figma/screenshot + spec.md → ui_contract.md                │
│  Gate: State machine completa + tokens Design System        │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 3: Mobile Architect                                  │
│  spec.md + ui_contract.md → arch.md + tasks.md              │
│  Gate: Sigue Clean Architecture + dependencias validadas    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 4: Code Agent (por vertical slice)                   │
│  arch.md + tasks.md → Código implementado                   │
│  Gate: Lint pass + tests pass + /security pass              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 5: PR Guardian                                       │
│  Código + spec + arch → PR listo para merge                 │
│  Gate: DoD completo + OWASP check + tests verdes            │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Definición de Agentes

---

### 3.1 Spec Agent

**Responsabilidad:** Transformar un ticket ambiguo en una especificación verificable y atómica.

**Cuándo se activa:** Al iniciar cualquier feature nueva o cambio significativo.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| Ticket URL o contenido | Jira / Linear / GitHub Issues | ✅ |
| Contexto de negocio | PM o documentación del producto | ✅ |
| Diseños (si existen) | Figma URL o screenshot | ⚠️ Opcional |
| API specs (si existen) | Swagger / OpenAPI | ⚠️ Opcional |

**Outputs generados:**

Todos los archivos se crean en `docs/features/FEAT-XXX/`:

- **spec.md** — Especificación completa con:
  - Resumen del feature (1-2 párrafos)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LIDR-academy/mobile-facephi](https://github.com/LIDR-academy/mobile-facephi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
