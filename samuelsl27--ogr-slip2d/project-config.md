---
trigger: always_on
description: Estabilidad de taludes por equilibrio límite, flujo subterráneo por
---

# OGR Slip2D

Estabilidad de taludes por equilibrio límite, flujo subterráneo por
elementos finitos y análisis probabilístico.

**Dos niveles de nombre, y conviene no confundirlos**: *OpenGeoRock* (u
*OGR Suite*) es el paraguas de cinco programas planificados; **OGR Slip2D**
es este programa, y es lo que contiene este repositorio. El paquete
instalable se llama `ogr-slip2d`. El núcleo compartido `ogr_core` vive aquí
por ahora, y se extraerá a su propio paquete cuando exista un segundo
programa que lo use — no antes.

Autor y titular del copyright: Samuel Sáez López (UPCT). Licencia
AGPL-3.0-or-later.

> Este archivo es el **contrato de trabajo** con cualquier agente de IA.
> Se consulta antes de cada acción. Si algo aquí contradice lo que parece
> razonable, gana este archivo — y si de verdad está mal, dilo antes de
> saltártelo.

---

## Stack

- **Lenguaje**: Python 3.11+, con anotaciones de tipo donde aclaren.
- **GUI**: PySide6 (Qt 6).
- **Geometría**: Shapely. **Numérico**: NumPy, SciPy.
- **Gráficas**: Matplotlib. **Informes**: reportlab. **CAD**: ezdxf.
- **Tests**: runner propio en `tests/_runner.py` (aporta un `pytest`
  simulado; **no** hay pytest real instalado).
- **Formato de proyecto**: `.ogr`, JSON puro.

## Comandos

```bash
QT_QPA_PLATFORM=offscreen python tests/_runner.py   # toda la suite
python -m ogr_gui                                    # abrir la aplicación
python -m ogr_cli --help                             # interfaz de terminal
pip install -e .                                     # instalar en editable
pip install -e ".[mcp]"                              # + servidor MCP (agentes)
python -m ogr_mcp --help                             # servidor MCP
```

`QT_QPA_PLATFORM=offscreen` es **obligatorio** para los tests: construyen
widgets Qt reales, solo que nunca llegan a una pantalla.

Durante el desarrollo se puede ejecutar solo una parte (desde v0.1.80):

```bash
python tests/_runner.py transient          # archivos que contengan eso
python tests/_runner.py transient seepage  # unión de los dos
python tests/_runner.py -k erfc            # solo tests con ese nombre
python tests/_runner.py --list transient   # enseña la selección, no ejecuta
```

El patrón admite fragmento, nombre de archivo o ruta —`transient`,
`test_transient_v130.py` y `tests/test_transient_v130.py` seleccionan lo
mismo— y no distingue mayúsculas. Un patrón que no case con nada **sale
con código 2**, para que una errata no se lea como una suite en verde.

**Una ejecución filtrada no es evidencia para publicar.** Lleva un aviso
`FILTERED RUN` antes y después de los totales precisamente por eso: antes
de una versión, la suite entera y sin argumentos.

## Estructura del proyecto

| Ruta | Contenido |
|---|---|
| `ogr_core/` | Geometría, materiales, cargas, soportes, proyecto, hidráulica, estadística, anotaciones, DXF |
| `ogr_slip2d/` | Motor LEM: 9 métodos, 7 búsquedas, rebanado, foco, optimización, retroanálisis |
| `ogr_fem2d/` | Elementos finitos: mallado y solvers de filtración |
| `ogr_gui/` | Interfaz PySide6: lienzo, ~30 diálogos, ventanas de interpretación, i18n |
| `ogr_cli/` | Interfaz de línea de comandos |
| `ogr_api/` | Capa de operaciones sin Qt (spec 008): *handles*, validación, deshacer, trabajos en subproceso, render PNG, `python_exec`. La usan el servidor MCP y un script; **no** importa PySide6, `ogr_gui` ni `mcp` |
| `ogr_mcp/` | Servidor MCP para agentes de IA (extra `[mcp]`, comando `ogr-slip2d-mcp`): una herramienta por operación de `ogr_api`, stdio y HTTP con token. Guías en `docs/mcp/` |
| `tests/` | Un archivo por área funcional |
| `docs/` | Planes, auditorías, changelog |
| `spec/` | Especificaciones SDD (constitución y features) |

Capas en un solo sentido: `ogr_core → ogr_slip2d / ogr_fem2d → ogr_api →
{ogr_gui, ogr_cli, ogr_mcp}`. Una regla que hoy solo impone la interfaz se
**mueve** a `ogr_core/project/rules.py` y la interfaz pasa a preguntarla;
copiarla es cómo acaban diciendo cosas distintas.

---

## Las siete reglas

Cada una existe porque su ausencia causó un problema real en este
proyecto. No son burocracia; son cicatrices.

### 1. El trabajo numérico se valida contra algo EXTERNO

Un método de análisis, un solver o una fórmula necesitan un **valor de
referencia**: un caso publicado, una solución cerrada o una identidad
analítica. **Nunca** una captura de lo que el código imprime hoy: un test
de instantánea consagra el bug.

Ejemplos de lo que sí cuenta, todos ya en la suite:

- el factor de seguridad de referencia de un caso conocido (los métodos
  LEM están validados así, con error < 0.7 %; los dos Corps of Engineers,
  contra las tablas dovela a dovela de la EM 1110-2-1902);
- una solución cerrada (el solver de filtración se contrasta con la
  respuesta escalón erfc y con las medias armónica y aritmética por
  capas);
- una identidad analítica (el retroanálisis se comprueba porque la fuerza
  activa y la pasiva deben coincidir **exactamente** con factor objetivo
  1.0);
- consistencia asintótica con un camino ya validado (el transitorio a
  tiempo grande debe reproducir el permanente).

### 2. Todo texto visible pasa por `tr()`

```python
from ogr_gui.i18n import tr
label = QLabel(tr("Number of slices:"))
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelsl27/OGR-Slip2D](https://github.com/samuelsl27/OGR-Slip2D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
