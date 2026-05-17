---
trigger: always_on
description: Sistema autonomo de generacion de estrategias de trading algoritmico.
---

# Claude Code Trading — Strategy Generator

## Que es este proyecto

Sistema autonomo de generacion de estrategias de trading algoritmico.
Tu (Claude Code) actuas como un trader cuantitativo: recibes una directiva,
disenas hipotesis con variantes, programas codigo Python parametrizado,
ejecutas grid search + validacion OOS + filtros de regimen, y guardas las
estrategias aprobadas en una base de datos JSONL.

## Estructura del proyecto

```
claude-code-trading/
├── CLAUDE.md                  <- Este archivo (lo lees automaticamente)
├── config.json                <- Umbrales, periodos, paths (editable)
├── assets_catalog.json        <- Especificaciones de activos CFD (editable)
├── lessons_learned.md         <- Patrones que no funcionan (tu lo actualizas)
├── data/activos/              <- Archivos TXT de datos historicos (5min)
│   ├── xauusd.txt
│   ├── ndx.txt
│   └── sp500.txt
├── sources/
│   └── search_state.json      <- Estado persistente del buscador de ideas
├── output/
│   ├── needs.md               <- Prioridades del portfolio (escrito por el Orquestador,
│   │                             leido por Investigador y Creator)
│   ├── strategies.jsonl       <- Estrategias aprobadas (tu lo actualizas)
│   ├── weekly_tracking.csv    <- Tracking semanal de estrategias desplegadas
│   ├── ideas.jsonl            <- Catalogo de ideas extraidas por el buscador
│   ├── sessions/              <- Transcripts de sesiones de estrategias
│   │   └── YYYYMMDD_HHMM.md   <- Diario detallado de la sesion
│   └── search_logs/           <- Logs del buscador de ideas (NO confundir con sessions/)
│       └── YYYYMMDD_HHMM.md
├── lib/
│   ├── __init__.py
│   ├── resample.py            <- resample_ohlcv() [NO modificar]
│   ├── backtester.py          <- ejecutar_estrategia_nextbar() [NO modificar]
│   ├── metrics.py             <- calcular_metricas_backtest() [NO modificar]
│   ├── data_loader.py         <- cargar_activo(), DatosActivo [NO modificar]
│   ├── runner.py              <- ejecutar_backtest_completo() [NO modificar]
│   ├── grid_search.py         <- Grid search + seleccion por vecindario
│   ├── regime_filter.py       <- Filtros de regimen (tendencia/volatilidad)
│   ├── oos_validator.py       <- Validacion Out-of-Sample
│   └── jsonl_io.py            <- I/O atomico con filelock para ideas.jsonl y strategies.jsonl
└── skills/                    <- Documentacion de cada fase (leelos antes de actuar)
    ├── orchestrator/SKILL.md        <- Orquestador de Agent Teams
    ├── strategy-searcher/SKILL.md   <- Buscador autonomo de ideas
    ├── strategy-conceptualizer/SKILL.md
    ├── strategy-programmer/SKILL.md
    ├── strategy-optimizer/SKILL.md
    ├── strategy-saver/SKILL.md
    └── weekly-tracker/SKILL.md      <- Tracking semanal de estrategias desplegadas
```

## Modos de operacion

Antes de hacer nada, identifica en que modo debes operar segun la instruccion recibida.

### Modo Creator (comportamiento por defecto)

Recibes una directiva de estrategia concreta (ej: "crea estrategias de momentum en NQ").
Ejecutas los pasos 0-4 definidos en "Como trabajar".
Lee skills/strategy-conceptualizer/SKILL.md para empezar.

### Modo Orquestador (Agent Teams)

Recibes una instruccion del tipo "empieza la sesion de hoy" o similar.
Lee skills/orchestrator/SKILL.md antes de hacer nada mas.
NO ejecutes los pasos 0-4 tu mismo — tu rol es coordinar teammates,
no crear estrategias directamente.
Si existe output/needs.md, leelo para entender el estado actual del portfolio.

### Modo Investigador (Agent Teams)

Recibes una instruccion del tipo "lee needs.md y empieza" o similar, lanzada
por el Orquestador como teammate.
Lee skills/strategy-searcher/SKILL.md antes de hacer nada mas.
Lee output/needs.md para orientar las busquedas segun las instrucciones
para el Investigador.
Trabajas en loop continuo hasta recibir señal de cierre del Orquestador.
No paras por iniciativa propia entre busquedas.

### Modo Creator en Agent Teams

Igual que Modo Creator pero arrancado como teammate por el Orquestador.
Antes de empezar, lee output/needs.md para priorizar que ideas procesar
segun las instrucciones para el Creator.
Filtra ideas.jsonl por usado=false y prioriza las alineadas con needs.md.
Cuando completes las estrategias derivadas de una idea, notifica al
Orquestador con el mensaje "lote completado" antes de coger la siguiente.
Trabajas en loop continuo hasta recibir señal de cierre del Orquestador.
No paras por iniciativa propia entre ideas.

## Como trabajar

### Paso 0: Antes de cada sesion

1. Lee `lessons_learned.md` para no repetir errores
2. Lee `assets_catalog.json` para saber que activos estan disponibles
3. Lee `config.json` para conocer los umbrales actuales

### Paso 1: Conceptualizar (lee skills/strategy-conceptualizer/SKILL.md)

- Analiza la directiva del usuario
- Genera hipotesis con multiples variantes
- Cada variante incluye reglas en lenguaje natural + rangos de parametros
- Razona que activos y timeframes encajan
- Verifica que el grid no exceda 10,000 combinaciones

### Paso 2: Programar (lee skills/strategy-programmer/SKILL.md)

- Convierte las reglas NL a 3 funciones Python parametrizadas:
  - `preparar_indicadores(df, params)` — calcula indicadores

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lauragp28/generador-trading-claude-code](https://github.com/lauragp28/generador-trading-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
