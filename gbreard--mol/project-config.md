---
trigger: always_on
description: **⛔ ANTES DE DEPLOYEAR: leer `DEPLOY_RULES.md` — producción es solo para Gerardo.**
---

# MOL - Monitor de Ofertas Laborales

## LEER PRIMERO: Estado Actual

**⛔ ANTES DE DEPLOYEAR: leer `DEPLOY_RULES.md` — producción es solo para Gerardo.**

**ANTES DE HACER CUALQUIER COSA, leer `.ai/learnings.yaml` para el estado actual del trabajo.**

El archivo `learnings.yaml` contiene:
- `current_state`: Qué estamos haciendo AHORA (ofertas, versiones, próximo paso)
- `ultimo_trabajo`: Qué se hizo en la última sesión
- `problemas_conocidos`: Issues actuales
- **`conteos`**: SINGLE SOURCE OF TRUTH para cantidades (reglas, skills, etc.)

**Comando del usuario: "Guardá estado"** → Ejecutar `python scripts/sync_learnings.py` + agregar notas en ultimo_trabajo si es necesario.

**AUTO-SYNC + REPORTE DE FASES (v2.0):** Al iniciar sesión, Claude recibe automáticamente:
- **Reporte de las 3 fases** con métricas actuales (ofertas, NLP, matching, validación)
- **Sugerencia de fase** basada en qué necesita atención (errores, pendientes, etc.)
- **Conteos actualizados** desde configs y BD

Triggers:
- **Al iniciar sesión** → Hook SessionStart (`.claude/settings.json`)
- Al ejecutar pipeline → `sync_learnings_yaml()` al final
- Manual: `python scripts/sync_learnings.py` (con `--human` para formato detallado)

---

## PENDIENTE CRITICO: Rotar Supabase service_role_key (S-01)

**La `service_role_key` actual está comprometida** (existe en el historial de git). El código ya fue limpiado (commit `9f904093`) pero la key vieja sigue siendo válida.

**Pasos a realizar (manual):**
1. Ir a **Supabase Dashboard → Settings → API** → regenerar `service_role_key`
2. Actualizar `config/supabase_config.json` local con la nueva key
3. Actualizar `SUPABASE_SERVICE_ROLE_KEY` en **Vercel** (si se usa)
4. Verificar que `sync_to_supabase.py` y `generate_mol_skills_profile.py` sigan funcionando
5. **Borrar esta sección de CLAUDE.md** una vez completado

---

## Descripcion
Sistema de monitoreo del mercado laboral argentino para OEDE. Scrapea ofertas de empleo, extrae informacion con NLP, clasifica segun taxonomia ESCO, y provee dashboards para analistas.

## Estado Actual

> **CONTEOS OFICIALES:** Ver `.ai/learnings.yaml` sección `conteos` (single source of truth)

- **NLP v11.4** (20 campos + source-aware pre-fill + postprocessor + qwen2.5:7b)
- **NLP Gate v1.1** (35+ reglas pre-matching, bloquea critico/alto)
- **Multi-Position Detection** (regex + LLM, crea sub-ofertas)
- **Matching v3.5.4 ESCO-FIRST** - ESCO es target, ISCO se deriva
- **Skills v2.5** - BGE-M3 base (LoRA fine-tuned NO disponible — model_lora no existe en disco, umbral 0.40)
- **Canonización planificada** - Reducir 14,247 skills ESCO a ~3,000 canónicas (ver `docs/plan/14_CANONIZACION_SKILLS_TAREAS.md`)
- **Pipeline v3.3** (8 pasos integrados con NLP Gate + multi-position)
- **Fine-tuning data** (llm_raw_json + postprocessor_diff + valor_actual/corregido)
- **Conteos dinámicos** (ver `learnings.yaml`): reglas_negocio, reglas_validacion, sinonimos_argentinos
- **Auto-sync** de learnings.yaml activado (v2.1)

### Matching v3.5.4 ESCO-First (2026-03-15)
```
PRINCIPIO: ESCO es el TARGET, ISCO es CONSECUENCIA

FLUJO DE PRIORIDAD:
1. REGLAS DE NEGOCIO (si aplican) → GANAN SIEMPRE
   - Buscan ocupación ESCO por label exacto
   - ISCO se deriva de la ocupación encontrada
   - titulo_original_contiene_alguno (v3.5.4: busca en título SIN limpiar)

2. DICCIONARIO ARGENTINO (si no hay regla)
   - Mapea términos argentinos a ISCO

3. SEMÁNTICO (fallback)
   - Skills + título embeddings (LoRA fine-tuned model)

METADATA GUARDADA:
- isco_semantico, score_semantico (siempre calculado)
- isco_regla, regla_aplicada (si aplica regla)
- dual_coinciden: 1=mismo, 0=difieren, NULL=solo semántico
- decision_metodo: "regla_prioridad" | "semantico_default"
```

### Fábrica de Procesamiento (Bloques I+G — 2026-03-22)

El procesamiento se organiza como una fábrica con dos líneas:

**LÍNEA 1 — FABRICACIÓN** (producir datos clasificados):
```
Scraping → NLP v11.4 → Gate NLP (35+ reglas) → Matching v3.5.4 → Gate Matching → Validación → Sync
```

**LÍNEA 2 — MEJORA CONTINUA** (mejorar la fábrica):
```
Errores → Issues → Training Pairs (602+) → Fine-tuning → Catálogo MOL → Perfil Argentino
```

Correcciones de L1 alimentan L2. Mejoras de L2 vuelven como reglas y modelos a L1.

**3 actores:** Pipeline (automático), Claude (operario calificado), Humano (gerente + QA)

**Admin UI** (`/admin/procesamiento/`):
- **Fábrica**: vista dual con control por nodo (botones ejecutar/reprocesar/configurar por etapa)
- **Diccionarios**: 6 configs editables en tabs (reglas, NLP, sinónimos, oficios, skills, limpieza)
- **Catálogo MOL**: curación skills/ocupaciones argentinas (detectada → revisión → catalogada)
- **Perfil Argentino**: publicación versionada del perfil consolidado
- **Validación**: estación del analista (OK/Error/Revisar/Basura + wizard + auto-issues)

**Gateway local**: tabla `pipeline_commands` en Supabase + poller local (mismo patrón que scraping_commands). Admin da órdenes desde UI sin terminal ni Claude.

**Configs editables desde UI** (override Supabase → fallback JSON local):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gbreard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
