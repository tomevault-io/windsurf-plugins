---
trigger: always_on
description: > Última actualización: 2026-03-31
---

# QTorres — Contexto para Claude Code

> Última actualización: 2026-03-31

## Reglas absolutas — NUNCA violar

Estas reglas son inviolables. No importa qué Issue estés implementando ni qué parezca razonable.
**Si alguna de tus acciones viola cualquiera de estas reglas, PARA y replantea.**

1. **NUNCA** poner faces nativas (`field`, `button`, `slider`) en el `pane` del canvas del editor. Renderizar TODO con Draw sobre `base`. (DT-026)
2. **PROHIBIDO** `do` con bloques dinámicos, `load` de strings, o `compose` en runtime del `.qvi` generado. El código generado debe compilar con `red -c`. (DT-028)
3. **Todo** en Red-Lang, sin excepciones. Sin dependencias externas. (DT-001)
4. **NUNCA** usar herencia profunda (A → B → C). Composición + prototipos + constructores siempre. (DT-023)
5. **NUNCA** implementar zoom en el canvas. (visual-spec 1.1)
6. **NUNCA** permitir múltiples wires a un puerto de entrada. (visual-spec 5.2)
7. **NUNCA** generar strings intermedios en el compilador. Siempre manipular bloques Red. (DT-008)
8. **NUNCA** empezar una fase sin completar la anterior. Respetar el orden del backlog.
9. **SIEMPRE** implementar dentro de los ficheros existentes en `src/`. No crear módulos nuevos sin aprobación explícita.
10. **SIEMPRE** ejecutar los tests (`red-cli tests/run-all.red`) tras cada cambio. No commitear con tests rotos.
11. **SIEMPRE** consultar el skill de Red-Lang (`skills/red-lang/SKILL.md`) antes de escribir código Red, especialmente Draw y View.
12. **SIEMPRE** respetar la separación de responsabilidades entre módulos (ver sección "Problemas conocidos de arquitectura").

## Qué es este proyecto

QTorres es una alternativa open source a LabVIEW construida íntegramente en Red-Lang. El usuario construye programas arrastrando bloques y conectándolos con wires, igual que en LabVIEW. Al guardar, QTorres genera un fichero `.qvi` con código Red/View completo que al ejecutarse muestra el Front Panel como una ventana, igual que LabVIEW.

**Nombre:** QTorres (por Torres Quevedo)
**Repositorio:** https://github.com/anlaco/QTorres
**Backlog:** https://github.com/users/anlaco/projects/1

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Lenguaje | Red-Lang 100% |
| UI del diagrama | Red/View + Draw dialect |
| UI del panel | Red/View |
| Compilador | Red puro (manipulación de bloques) |
| Formato de fichero | Sintaxis Red nativa |

Sin dependencias externas. Un solo binario. Funciona en Linux, Windows y macOS.

## Estructura del proyecto

```
QTorres/
├── CLAUDE.md               # Este fichero — contexto principal para IA
├── README.md
├── docs/
│   ├── arquitectura.md     # Arquitectura de módulos
│   ├── plan.md             # Plan por fases
│   ├── decisiones.md       # Decisiones técnicas (DT-001 a DT-029)
│   ├── PLANNING.md         # Decisiones pendientes críticas
│   ├── retos.md            # Riesgos y dificultades
│   ├── visual-spec.md      # Especificación visual (documento vivo)
│   ├── tipos-de-fichero.md # Mapeo LabVIEW → QTorres
│   ├── labview-comportamiento.md # Arquitectura LabVIEW: renderizado, modos, estilos
│   └── GTK_ISSUES.md       # Bugs del backend GTK en Linux
├── src/
│   ├── qtorres.red         # Punto de entrada + toolbar + ventana principal
│   ├── graph/
│   │   ├── model.red       # Modelo: make-label, make-node, make-wire, make-fp-item, set-config, find-node-by-id (635 líneas)
│   │   └── blocks.red      # Registro de bloques + dialecto block-def — 40 bloques
│   ├── compiler/
│   │   └── compiler.red    # Compilador: topo-sort, bind-emit, compile-body/diagram/panel (1165 líneas)
│   ├── runner/
│   │   └── runner.red      # Runner: ejecución en memoria con do
│   ├── io/
│   │   └── file-io.red     # File I/O: serialize, format, save/load .qvi, save/load-panel (738 líneas)
│   └── ui/
│       ├── diagram/
│       │   ├── canvas.red          # BD canvas: hit-test, CRUD, actor render-diagram (1233 líneas)
│       │   ├── canvas-render.red   # Render puro BD: constantes, geometría, Draw (934 líneas)
│       │   └── canvas-dialogs.red  # Diálogos edición, paleta, SR helpers (413 líneas)
│       └── panel/
│           ├── panel.red           # FP: hit-test, diálogos, paleta, actor render-panel (570 líneas)
│           └── panel-render.red    # Render puro FP: constantes, Draw, waveform (411 líneas)
├── tests/
│   ├── run-all.red         # Runner de tests automatizados
│   ├── test-blocks.red     # Tests del registro de bloques (40 bloques, puertos, emit)
│   ├── test-topo.red       # Tests de topological sort (lineal, diamante, vacío, ciclos)
│   ├── test-model.red      # Tests del modelo (make-node, make-wire, make-frame, make-structure)
│   └── test-compiler.red   # Tests del compilador (bind-emit, compile-body, round-trip, estructuras)
├── examples/
│   ├── suma-basica.qvi         # Ejemplo de VI simple
│   ├── while-loop-basico.qvi   # While Loop básico
│   ├── while-loop-suma.qvi     # While Loop con shift registers
│   ├── for-loop-basico.qvi     # For Loop (suma 0..9 = 45)
│   ├── case-numeric.qvi        # Case Structure con selector numérico
│   ├── case-boolean.qvi        # Case Structure con selector booleano (either)
│   ├── suma-subvi.qvi          # Ejemplo de sub-VI
│   └── programa-con-subvi.qvi
├── .github/workflows/      # CI: tests automáticos en push/PR a main

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anlaco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
