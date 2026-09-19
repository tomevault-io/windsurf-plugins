---
trigger: always_on
description: Repo de **desarrollo y validación de estrategias de trading con el método TIS**, ejecutado por
---

# CLAUDE.md — MINI FONDO

Repo de **desarrollo y validación de estrategias de trading con el método TIS**, ejecutado por
4 agentes encadenados en un harness sobre un motor de validación único (`codigo/quantlab/`).
Es el lead magnet de Trade It Simple: el que lo reciba tiene que poder clonarlo, correr el
ejemplo y meter su propia hipótesis sin preguntar nada. Escribe para esa persona.

**La audiencia no es técnica.** La puerta de entrada es `EMPIEZA_AQUI.md` (guía por niveles, sin
jerga) y `EMPEZAR.bat` / `empezar.sh` (puesta en marcha con doble clic). El `README.md` es la
referencia técnica, no el primer contacto. Cualquier cosa que se añada tiene que caber en uno de
los tres niveles: **0** mirar sin instalar, **1** correr el motor, **2** los agentes.

## Los agentes (`.claude/agents/`)
| Agente | Pasos TIS | Entra | Sale | Veredicto |
|---|---|---|---|---|
| `investigador` | 01-02 | `hipotesis.md` | `informe_aed.md`, `codigo/exploratorio_<ID>.py` | `EDGE` / `NO_EDGE` |
| `protocolo` | 03 | hipótesis + AED | `reglas.md` | `OK` |
| `motor` | 04-07 | `reglas.md` | `codigo/estrategias/<carpeta>.py`, `reportes/<carpeta>/`, `informe_motor.md` | `OK` |
| `validador` | cierre + 08 | todo lo anterior | `informe_validacion.md`, `checklist_deploy.md` | `APROBADA` / `RECHAZADA` |
| `eficiencia` | transversal | estado, bitácora, entregables | `eficiencia.md` (notas para el siguiente agente) | `FLUIDO` / `AVISO` / `BLOQUEADO` |
| `mariel` | fuera del grafo | lo que le preguntes | una respuesta, no ficheros | — (no mueve fase) |

`mariel` es la mentora del método: revisa hipótesis, le busca el hueco a un resultado, traduce los
informes y avisa cuando alguien se está haciendo trampa. **Solo lee** (sin Write ni Bash) a
propósito: no produce entregables. No confundir con la autoría — quien opera el repo es la persona
que lo recibió, y los otros cinco agentes le hablan a ella, nunca a "Mariel".

`eficiencia` no está en el grafo: corre solo después de cada fase de agente (se apaga con
`--sin-eficiencia`) o a petición con `python -m harness.run eficiencia <ID>`. No mueve la fase, no toca
entregables, no decide criterio.

Los mismos ficheros sirven para dos cosas: el harness los usa como system prompt, y desde
Claude Code se invocan como subagentes (`@investigador`, etc.) para trabajo interactivo.

## El grafo (`harness/grafo.py`)
```
investigacion -EDGE-> [puerta_hipotesis] -ok-> protocolo -> motor -> validacion -APROBADA-> [puerta_deploy] -ok-> incubacion
      |NO_EDGE              |no                              ^         |RECHAZADA (<=3 vueltas)         |no
   archivada             archivada                           +---------+  4a vez -> archivada        archivada
```
Las `[puertas]` las cierra la persona (`ok`/`no`). Son los pasos verdes: criterio.
Todo lo demás lo cierra un agente con su línea `VEREDICTO:`.

## El método (`.claude/skills/` + `docs/`)
Los seis skills del método TIS están instalados en `.claude/skills/tis-*/SKILL.md`: `tis-estilo`
(gobierna a los otros cinco: voz, anti-slop y no negociables), `tis-research-edge`,
`tis-diseno-estrategia`, `tis-validacion`, `tis-riesgo-portafolio` y `tis-bitacora-estrategia`.
**Aplica `tis-estilo` a cualquier texto que escribas en este repo.** Lo explicado para humanos está
en `docs/METODO_TIS.md`.

Los seis agentes declaran `Skill` en su frontmatter `tools:`. Es obligatorio: `grafo.py` pasa
`allowed_tools=spec["tools"]` tal cual, así que un agente sin esa herramienta **no puede cargar los
skills** y la instrucción "aplica tis-estilo" se queda en papel mojado. Si añades un agente, no te
olvides de `Skill`.

La autoridad de criterio es **`docs/MIS_REGLAS.md`**, el fichero de la persona: sus umbrales mandan
sobre los defaults de `docs/PROTOCOLO.md` y sobre `quantlab.validation.Criterios`. Ningún agente lo
edita.

## El motor (`codigo/quantlab/`)
Un solo motor para todas las estrategias. `backtest.py` ejecuta a la apertura siguiente con stop
ATR intrabarra y costes por lado; `validation.py` corre las 5 fases (IS/OOS, walk-forward, meseta,
Monte Carlo, stress); `report.py` escribe `reportes/<carpeta>/RESUMEN.md`. Una estrategia es
`codigo/estrategias/<ID>_<nombre>.py` con `senal(df, **params)` (≤30 líneas) y un dict `PLAN`.
`codigo/validar.py <ID> <datos>` lo corre todo. **No se escribe un motor por estrategia.**

## Origen
Fusión (2026-09-16) de tres sesiones en vivo del 2026-09-15: el laboratorio `quant_lab`
(motor, docs, Kaufman y Raschke → 001-004), el repo de agentes `CUEVA` (agentes, harness, dashboard,
pruebas → 005-007) y el agente de eficiencia. Este repo es la única copia viva.

## Cómo se usa
```
.venv\Scripts\activate
python -m pytest -q
python codigo/validar.py 001 --sintetico --rapido      # placebo
python codigo/validar.py 001 data/NDX_D1.csv           # ejemplo con datos
python -m harness.run nueva 002 nombre                 # carpeta + plantilla de hipótesis
python -m harness.run run 002                          # corre hasta la siguiente puerta
python -m harness.run ok 002 | no 002                  # cierras la puerta
python -m harness.run status
python -m harness.run eficiencia 002                 # revisión a petición
streamlit run harness/dashboard.py
```

## Dónde va cada cosa
| Qué | Dónde |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlangsfinance-stack/MiniFondoCueva](https://github.com/mlangsfinance-stack/MiniFondoCueva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
