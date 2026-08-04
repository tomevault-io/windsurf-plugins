---
trigger: always_on
description: >
---


# AGENTS.md — Guía de Trabajo para Agentes de IA

> **Audiencia:** Agentes de IA, Claude Code, GitHub Copilot, colaboradores humanos.
> **Modo de lectura:** Secuencial obligatorio para primera visita; por sección para consultas puntuales.
> **Cardinalidad:** Este documento es la **fuente de verdad canónica** para todas las reglas de ingeniería del repositorio.
> **Entrypoint rápido:** Si solo necesitas orientarte, comienza por [`SOURCE_OF_TRUTH.md`](./SOURCE_OF_TRUTH.md).

Este documento define cómo trabajar correctamente en el repositorio `chile-hub`.
Es la fuente de verdad para cualquier agente de IA o colaborador nuevo que necesite
entender la arquitectura, las reglas no negociables y las convenciones del proyecto.

> **Leer completo antes de modificar cualquier archivo.**
> Las reglas de esta guía evitan errores que se propagan silenciosamente a los datos publicados.

---

## 1. Propósito del proyecto

`chile-hub` es una capa de datos pública, curada y reproducible sobre **datos oficiales de Chile**.
Actualmente publica diecinueve (<!-- START_AGENTS_DATASET_COUNT -->22<!-- END_AGENTS_DATASET_COUNT -->) capas:

| Capa | Fuente | Descripción |
|:---|:---|:---|
| **División Político-Administrativa** (regiones, provincias, comunas) | BCN ArcGIS | 16 regiones, 56 provincias, 346 comunas con códigos CUT, coordenadas y abreviaturas |
| **Comunas Enriquecidas** | BCN ArcGIS + INE | Comunas con coordenadas de cabecera y población estimada INE, listas para análisis territorial |
| **Indicadores Económicos** | mindicador.cl (datos BCCh / INE) | UF, Dólar, Euro, UTM, IPC — histórico desde 2010, actualización diaria |
| **Censo Comunal 2024** | INE | Población por sexo y cinco grandes grupos de edad para las 346 comunas |
| **Censo Hogares y Viviendas 2024** | INE | Viviendas y hogares por comuna, incluyendo promedios de personas por hogar |
| **Establecimientos de Salud** | MINSAL / datos.gob.cl | Directorio vigente con tipo, dependencia, urgencia, estado y coordenadas |
| **Distritos Electorales** | BCN / SERVEL | Asociación de comunas a distritos electorales de diputados y circunscripciones senatoriales |
| **Establecimientos Educacionales** | MINEDUC | Directorio oficial con RBD, dependencia, ubicación y estado de funcionamiento |
| **Finanzas Municipales** | SINIM / SUBDERE | Indicadores financieros municipales anuales por comuna |
| **Resultados Educacionales** | MINEDUC | Métricas educacionales agregadas por comuna y año, sin registros personales |
| **Indicadores Urbanos SIEDU** | INE / SIEDU | Indicadores urbanos en formato largo con cobertura parcial esperada |
| **Perfil Territorial Comunal** | chile-hub derivado | Una fila por comuna con métricas territoriales consolidadas |
| **Empresas (RES)** | Ministerio de Economía / datos.gob.cl | Registro de constituciones de empresas bajo Ley 20.659 con RUT, razón social, tipo societario y comuna |
| **Pobreza Comunal (SAE)** | MDS / Observatorio Social | Estimaciones de pobreza por ingresos y multidimensional por comuna |
| **Consumo Eléctrico Comunal** | CNE / Energía Abierta | Consumo eléctrico anual por comuna y tipo de cliente |
| **Partidos Políticos** | Cámara de Diputados / SERVEL | Roster de partidos políticos vigentes e históricos con estado legal |
| **Autoridades Electas** | Cámara de Diputados + Senado | Diputados y senadores en ejercicio, con partido y distrito o circunscripción |
| **Delincuencia Comunal** | CEAD / Subsecretaría de Prevención del Delito | Casos policiales DMCS y otras categorías por comuna y mes (carril `candidate`, ver nota abajo) |
| **Autoridades Locales** | BCN SIIT + Wikipedia (CC BY / CC BY-SA) | Gobernadores regionales (Wikipedia) y alcaldes (BCN SIIT, 100% cobertura), segregado de Autoridades Electas por licencia mixta (carril `candidate`) |

**El objetivo no es tener todos los datos de Chile. Es entregar un número pequeño de datasets
limpios, versionados, validados y consumibles en una línea de código.**

> **Carriles de publicación:** no todos los datasets listados arriba están en el bundle
> público. Algunos viven en el carril `candidate` (evaluados, implementados, pero fuera
> del ZIP publicable por fragilidad de fuente o licencia) con una fecha `review_by` de
> reevaluación. La fuente de verdad de qué dataset está en qué carril, su
> `maturity_status` y `confidence_tier` es **`data/dataset_catalog_config.json`** (y
> `data/source_registry.json`); los criterios de aceptación completos viven en
> **`docs/dataset-inclusion-criteria.md`**. No dupliques ese sistema aquí — referencia
> esos documentos.

---

## 2. Estructura del repositorio

```
chile-hub/
├── .github/workflows/
│   └── pipeline-check.yml      CI/CD: extrae, construye, valida, publica
│
├── src/
<!-- START_AGENTS_EXTRACTOR_LIST -->
│   ├── extractors/                 19 extractores por dataset + 4 módulos compartidos (ver nota abajo)
│   │   ├── base.py                                       BaseExtractor ABC (contrato para todos los extractores)
│   │   ├── http_utils.py                                 Reintentos/backoff HTTP compartidos
│   │   ├── region_utils.py                               Normalización de nombres de región compartida
│   │   ├── source_adapter.py                             Adaptador de fuente compartido

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cortega26/chile-hub](https://github.com/cortega26/chile-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
