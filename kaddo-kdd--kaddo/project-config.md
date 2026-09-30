---
trigger: always_on
description: Prompts reutilizables que convierten context packs en conocimiento del proyecto.
---


```bash
kaddo add agents
```

Los prompt packs de agentes son prompts en Markdown versionables que usas **en tu chat LLM
favorito** (Claude, ChatGPT, Cursor, Copilot, Windsurf…). Convierten un context pack de
Kaddo en conocimiento estructurado del proyecto.

> **Kaddo no ejecuta estos agentes.** El CLI prepara contexto determinista; el LLM hace la
> interpretación. Sin API key, sin proveedor de modelo, sin automatización.

## Agentes por momento de operación

Cada agente interviene en uno de los [momentos de operación](/es/operating-moments/) de Kaddo:

| Momento | Agentes |
|---|---|
| **Base** | bootstrap-agent · business-agent · codebase-agent |
| **Definición** | business-agent · product-agent · capability-agent · codebase-agent · architecture-agent · adr-agent/decision-agent |
| **Proyección** | roadmap-agent · backlog-agent · work-item-agent · ownership-agent |
| **Ejecución** | implementation-agent · ownership-agent · architecture-agent · capability-agent · adr-agent · guard-agent · capsule-agent · graph-agent |

## Instalación

`kaddo add agents` crea `knowledge/agents/`:

Los agentes se instalan en **carpetas por capa**:

```
knowledge/agents/
  README.md
  business/   business-agent.md
  product/    bootstrap-agent.md · capability-agent.md
  tech/       architecture-agent.md · codebase-agent.md · stack-agent.md ·
              security-agent.md · standards-agent.md · module-design-agent.md · adr-agent.md · capsule-agent.md · graph-agent.md
  delivery/   backlog-agent.md · roadmap-agent.md · work-item-agent.md · implementation-agent.md · ownership-agent.md · git-strategy-agent.md
  utilities/  legacy-agent.md
```

Los archivos existentes nunca se sobrescriben en silencio — al re-ejecutar solo se instalan
los que falten. `kaddo init` **no** instala agentes; agrégalos cuando los necesites.

## Instalación progresiva y grupos de agentes

Los agentes se instalan **progresivamente**, por capa — no obtienes todos de golpe. Por
defecto `kaddo add agents` instala solo el conjunto recomendado para el estado del proyecto:

| Estado | Instala |
|---|---|
| `new` | business-agent · bootstrap-agent · codebase-agent · roadmap-agent · backlog-agent · work-item-agent · implementation-agent |
| `pre-ai` | capability-agent · architecture-agent · roadmap-agent · backlog-agent · work-item-agent · implementation-agent |
| `legacy` | legacy-agent · architecture-agent · capability-agent · roadmap-agent · backlog-agent · work-item-agent · implementation-agent |

Los agentes se organizan en **grupos** por capa:

| Grupo | Agentes |
|---|---|
| `business` | business-agent |
| `product` | bootstrap-agent · capability-agent |
| `tech` | architecture-agent · codebase-agent · stack-agent · security-agent · standards-agent · module-design-agent · adr-agent · capsule-agent · graph-agent |
| `delivery` | backlog-agent · roadmap-agent · work-item-agent · implementation-agent · ownership-agent · git-strategy-agent |
| `utilities` | legacy-agent |

```bash
kaddo add agents                 # conjunto recomendado para el estado del proyecto
kaddo add agents --all           # todos los agentes
kaddo add agents --group tech    # un grupo de capa
```

`kaddo understand` reporta tu **fase actual** a partir del estado real del conocimiento —
Discovery → Planning → Delivery Preparation → Active Delivery → Maintenance — y recomienda el
agente para esa fase (ver [understand](/es/commands/understand/#recomendaciones-según-el-estado-real)).

## Agentes de entendimiento

| Agente | Propósito | Guarda en |
|---|---|---|
| `capability-agent` | Extraer/proponer capacidades del sistema | `knowledge/product/capabilities.md` |
| `architecture-agent` | Reconstruir el baseline de arquitectura | `knowledge/tech/current-state.md` |
| `roadmap-agent` | Proponer candidatos de roadmap | `knowledge/delivery/roadmap.md` |
| `legacy-agent` | Detectar riesgos/incógnitas antes de tocar código legacy | `knowledge/legacy/*.md` |
| `adr-agent` | Proponer decisiones de arquitectura candidatas | `knowledge/tech/decision-candidates.md` |

## Agentes de bootstrap

Para proyectos nuevos, refinan la base de conocimiento creada por
[`kaddo bootstrap`](/es/commands/bootstrap/) en las capas Business → Product → Tech → Delivery.

| Agente | Propósito | Guarda en |
|---|---|---|
| `business-agent` | Convertir una idea en definición de negocio | `knowledge/business/*.md` |
| `bootstrap-agent` | De negocio a capacidades, atributos de calidad y roadmap | `knowledge/bootstrap-summary.md`, `capabilities.md`, `roadmap.md` |
| `codebase-agent` | Proponer una base de codebase (sin código) | `knowledge/tech/codebase.md` |

## Agentes operativos

Apoyan la ejecución diaria y los artefactos multirepo / globales (VS-017).

| Agente | Propósito | Guarda en |
|---|---|---|
| `backlog-agent` | Capturar ideas/notas crudas en un draft o candidato de roadmap (sin refinar) | `knowledge/delivery/work-items/draft/` o un candidato de roadmap |
| `work-item-agent` | Redactar y refinar un work item desde el contexto | work item activo |
| `implementation-agent` | Implementar un Work Item refinado; sugerir branch/scan/owners/guard | código · tests · conocimiento actualizado |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaddo-kdd/kaddo](https://github.com/Kaddo-kdd/kaddo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
