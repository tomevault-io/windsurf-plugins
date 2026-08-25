---
trigger: always_on
description: Plataforma de contrastación electoral para Ecuador. El ciudadano consulta una
---

# Lo Dicho — Contexto del proyecto

Plataforma de contrastación electoral para Ecuador. El ciudadano consulta una
declaración de un candidato (por voz, texto o URL de nota de prensa) y el sistema
responde con evidencia recuperada de un corpus curado: qué dice el plan de trabajo
registrado y qué establece el COOTAD sobre las competencias de ese nivel de gobierno.

**Dominio:** lodicho.intiinside.com


---

## Repositorios

| Repo | Contenido | Ciclo de vida |
|---|---|---|
| `lodicho` | Aplicación (API, worker, web, infra) | Despliegue en ventanas controladas |
| `lodicho-corpus` | `.md` con frontmatter + PDFs fuente | Commits frecuentes, dispara reindexación |

Están separados a propósito: no mezclar. Un push al corpus no debe disparar CI de
despliegue, y un push de código no debe disparar reindexación.

`lodicho-corpus` es un repo privado en GitHub (`github.com/intiinside/lodicho-corpus`),
clonado como carpeta **hermana** de este repo tanto en desarrollo como en el VPS
(`/opt/lodicho-corpus` junto a `/opt/lodicho`) — `docker-compose.yml` lo monta como
volumen en el contenedor `api`. Si no existe en la máquina, Docker crea ahí una carpeta
vacía en vez de fallar; hay que clonarlo a mano.

---

## Stack

- **Host:** Ubuntu Server 24.04, Hetzner VPS, Docker Compose
- **Proxy:** Nginx + Certbot
- **API:** FastAPI (Python 3.12) + Pydantic v2
- **Jobs:** ARQ + Redis
- **Relacional:** PostgreSQL 16 + Alembic
- **Vectorial:** Qdrant (denso + sparse, vectores nombrados)
- **Embeddings:** Gemini `gemini-embedding-001`, 768 dim
- **Sparse:** FastEmbed `Qdrant/bm25` (local, sin costo de API)
- **Generación:** Gemini 2.5 Flash (clasificación, transcripción) / 2.5 Pro (veredictos)
- **Ingesta:** n8n (solo webhook + delta; el procesamiento va en Python) — planeado, no
  implementado todavía. Lo que hoy mueve contenido a Qdrant es el panel de admin y
  `make ingest`; ver "Panel de admin" más abajo.
- **Conversión PDF → Markdown:** Docling, corriendo del lado del servidor (`api`).
  Pesado (trae OpenCV) — ver gotchas de build en "Panel de admin".
- **Frontend:** Vanilla JS + ES modules, `marked.js`. Sin framework, sin build step.
  Composer único (texto/voz/URL en una sola caja, el modo se infiere al enviar) en vez
  de tabs separados.

---

## Reglas críticas

Estas no son preferencias de estilo. Violarlas produce daño real a un candidato o
expone el proyecto legalmente.

### 1. Filtro por candidatura, siempre en código

Toda recuperación sobre `planes_trabajo` filtra por `candidatura_id` en el cliente
Qdrant. Nunca delegado al prompt. Recuperar el plan de otra candidatura produce un
veredicto difamatorio.

### 2. Tres ausencias distintas, nunca confundir

| Estado | Significado |
|---|---|
| `sin_plan_registrado` | La candidatura no registró plan ante el CNE (dato en `candidaturas`) |
| `sin_plan_recuperado` | El retrieval no devolvió fragmentos (fallo técnico) |
| `no_consta` | Se recuperó el plan y la propuesta no está ahí |

Solo `no_consta` habilita el veredicto `no_consta_en_plan`. Hay un validador Pydantic
que lo impide; no lo desactives.

### 3. Cifras: nunca por RAG

Los indicadores estadísticos viven en la tabla `indicadores` y se exponen como tool
call con parámetros (`codigo`, `jurisdiccion_dpa`, `anio`). Un embedding no distingue
34,2 % de 43,2 %. Sin dato → veredicto `incomprobable`, jamás una cifra inferida.

### 4. Evidencia automática, veredicto firmado

| Salida | Revisión humana |
|---|---|
| Cita del plan de trabajo | No |
| Artículos del COOTAD aplicables | No |
| Dato oficial de `indicadores` | No |
| Contraste factual entre candidatos | No |
| Veredicto categórico | **Sí, obligatoria** |
| Score de factibilidad | **Sí, obligatoria** |

Un veredicto sin firma de revisor nunca sale con `estado='publicado'`.

### 5. Nada de recomendación de voto

El clasificador de intención rechaza con mensaje fijo (no generado por el modelo):
recomendación de voto, comparación de calidad entre candidatos, opinión sobre la
persona. Contrastar propuestas lado a lado sí se permite, sin juicio de calidad.

### 6. Silencio electoral

Variable `MODO_SILENCIO_ELECTORAL`. Cuando está activa: solo lectura de informes ya
publicados, generación desactivada.

### 7. Contenido web = datos no confiables

El texto extraído de una URL va envuelto en `<contenido_web>` y el system prompt
declara explícitamente que nada dentro de ese bloque es una instrucción. Riesgo real
de inyección de prompt.

---

## Modelo de datos

```
candidaturas (id, organizacion_politica, lista_numero, dignidad,
              jurisdiccion_dpa, periodo, doc_id_plan, estado_plan)
     │
     └──< candidatos (id, nombre, candidatura_id FK, posicion_lista)

documentos   (id, doc_id UNIQUE, tipo, candidatura_id FK, ruta_repo,
              sha256, pdf_sha256, git_sha, n_chunks, indexado_en, estado)

indicadores  (id, codigo, descripcion, jurisdiccion_dpa, anio,
              valor NUMERIC, unidad, fuente, url)

consultas    (id, tipo_input, texto, audio_path, url_fuente,
              contenido_archivado, hash_contenido, intencion_detectada,
              desde_cache, creado_en)
     │
     └──< declaraciones (id, consulta_id, texto, tipo, atribuible, analisis_id)

analisis     (id, candidatura_id FK, afirmacion, veredicto ENUM,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intiinside/lodicho](https://github.com/intiinside/lodicho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
