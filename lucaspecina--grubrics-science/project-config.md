---
trigger: always_on
description: Entrena Qwen3-8B con RL (GRPO) para generar rúbricas de evaluación médica y científica. La señal de reward es *functional alignment*: correlación de Spearman entre los rankings del Judge (GPT via Azure) y los gold_scores de médicos/expertos humanos.
---

# GRubrics

Entrena Qwen3-8B con RL (GRPO) para generar rúbricas de evaluación médica y científica. La señal de reward es *functional alignment*: correlación de Spearman entre los rankings del Judge (GPT via Azure) y los gold_scores de médicos/expertos humanos.

## Stack

- **Modelo**: Qwen3-8B + LoRA (rank 64)
- **RL framework**: veRL (GRPO) | **SFT**: TRL + LoRA
- **Rollout**: vLLM (H100)
- **Judge**: GPT-4.1 via Azure OpenAI, scoring binario a-la-HealthBench (1 call/criterion, pass/fail) — CHG-021
- **Tracking**: wandb | **Env**: `conda activate RL`

## Workflow de desarrollo

- **Desarrollo local**: Windows — editar código, leer logs, planear experimentos
- **Ejecución**: H100 remota (Linux, Azure ML) — training, precompute, baselines
- **SSH directo**: Claude puede ejecutar comandos via `ssh azure-ml "comando"` (key auth, sin password)
- **Env de training**: `conda activate RL` (siempre, en la H100)
- **Dinámica**: editar local → push → Claude ejecuta en H100 via SSH (o el usuario reporta)
- **Nunca asumir** que un comando se puede ejecutar localmente — preguntar siempre si hay duda
- **Setup de la VM**: ver `docs/h100-setup.md` para reproducir el entorno desde cero

## Los tres actores

- **GRubrics** (Qwen3-8B + LoRA) — se entrena, genera rúbricas
- **Judge** (GPT fijo) — evalúa respuestas con la rúbrica generada
- **Answer Policy** (GPT fijo) — generó las respuestas pre-computadas (offline, en `data/cache/`)

## Convenciones del repo

- Configs en `configs/` — GRPO: `verl_grpo.yaml`, SFT: `sft_healthbench.yaml`
- Presets de datos: `configs/training_presets.yaml` (`open_only` default, `verifiable_only`, `curriculum`, `full_mix`)
- Checkpoints: `checkpoints/grubrics-transfer/`
- Cache precompute: `data/cache/*.jsonl` — **NO borrar**, cada run cuesta $
- Vars de entorno: `AZURE_API_BASE`, `AZURE_API_KEY`, `AZURE_API_VERSION`, `RUBRIC_JUDGE_MODEL`, `REWARD_LAMBDA_*`
- Tests: `pytest tests/ -v` (181 tests, todos deben pasar antes de commitear)

## Dónde está cada cosa

- `grubrics_science/rewards/grubrics_reward.py` — reward function unificada (async)
- `grubrics_science/judge/judge.py` — Judge async con rate limiting, retry, cache
- `grubrics_science/data/adapters/` — 7 adapters (healthbench, medqa, medmcqa, gsm8k, math, frontierscience)
- `run_sft.py` / `run_grpo.py` — launchers principales
- `notebooks/analyze_rubrics.ipynb` — análisis post-training
- `scripts/` — download_datasets, run_baselines, validate_judge, analyze_precompute
- `scripts/validate_e2e_pipeline.py` — validación E2E completa: SFT→GRPO→Resume (~35 min en H100)
- `docs/h100-setup.md` — guía para reproducir el entorno de la H100 desde cero

## Guardrails de evaluación y testing

Reglas para evitar falsos negativos cuando se evalúan modelos, judges, o componentes del pipeline:

1. **Timeout generoso siempre**: usar `--timeout 300` (mínimo) en cualquier script que llame a la API. Los modelos de reasoning (gpt-5-mini, o1, etc.) pueden tardar >120s. Los modelos estándar (gpt-4.1, gpt-4o) también pueden ser lentos en prompts largos. **Nunca usar el default sin verificar cuál es.**

2. **Guardar output siempre**: usar `--output` para guardar resultados detallados. Sin output no se puede diagnosticar si un resultado malo fue por el modelo o por un error de infraestructura (timeout, parse failure, rate limit).

3. **Verificar scores sospechosos**: accuracy=0.000 o kappa=0.000 son señales de rotura, no de modelo malo. Antes de concluir que un modelo "no sirve", revisar:
   - ¿Cuántas entries fueron evaluadas vs skipped?
   - ¿Cuántas tuvieron parse failure (score=0.0 por default)?
   - ¿Los raw responses del modelo tienen sentido?

4. **Parse failures silenciosos**: `judge.py:_parse_response` devuelve `[0.0]` cuando el JSON no parsea. Esto NO se reporta como error — la entry cuenta como evaluada con score 0. Revisar warnings del logger.

5. **Reproducir antes de concluir**: si un resultado contradice lo esperado (ej: modelo que funciona para todos menos para nosotros), repetir con parámetros controlados antes de documentar como hallazgo.

**Ref**: CHG-020 (timeout fix), EXP-JUDGE-001 (resultados GPT-4.x posiblemente artefacto).

## Comportamientos conocidos de veRL

Estos no son bugs sino comportamientos del framework que hay que tener en cuenta:

- **veRL JSON columns**: parche auto-aplicado al cargar datos en `rl_dataset.py`
- **Judge cache en RL**: siempre `max_cache_size=0` durante training (RAM unbounded si no)
- **veRL auto-resume + total_training_steps absoluto**: veRL detecta checkpoints en `default_local_dir` y resume automáticamente. `total_training_steps` es absoluto (no relativo al checkpoint). Borrar el directorio de checkpoints antes de un run from scratch con pocos steps.
- **veRL checkpoints**: guarda 3 formatos en cada step: FSDP shard (`model_world_size_*.pt`), HuggingFace (solo config+tokenizer en `huggingface/`), LoRA adapter (`lora_adapter/`). Resume usa el FSDP shard, no HF format.
- **Checkpoint save time**: ~150-185s por step (~80% del step time con batch=4). No es un bug — usar `save_freq` alto en producción.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucaspecina/grubrics-science](https://github.com/lucaspecina/grubrics-science) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
