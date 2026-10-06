---
trigger: always_on
description: Plataforma FEM en el navegador (C++/Eigen → WebAssembly) con análisis modal, Winkler springs nativos en C++, ejemplos parametrizados (zapatas, placas, cáscaras, pórticos, edificios) y un **workspace unificado** con Tweakpane para cargar cualquier ejemplo por selector o URL `?t=<id>`.
---

# Hekatan Struct Lineal

Plataforma FEM en el navegador (C++/Eigen → WebAssembly) con análisis modal, Winkler springs nativos en C++, ejemplos parametrizados (zapatas, placas, cáscaras, pórticos, edificios) y un **workspace unificado** con Tweakpane para cargar cualquier ejemplo por selector o URL `?t=<id>`.

**Deploy público:** https://giorgioburbanelli89.github.io/hekatan-struct-lineal/workspace/

## Arquitectura del workspace

**Un solo entorno: el workspace.** Cada ejemplo vive en su carpeta (`examples/src/<id>/<name>.ts`) y **exporta un `ExampleDef`** que el workspace registra en `exampleRegistry.ts`. No hay páginas por ejemplo: el build compila el workspace (y `fem-explained`, la única lección que aún trae su interfaz) y deja una redirección en cada dirección de antes (`PAGINAS_DE_ANTES` en `examples/vite.config.ts`).

```
examples/src/
├── workspace/                  ← hub principal
│   ├── main.ts                 (Tweakpane unificado: selector + params + modal)
│   ├── exampleRegistry.ts      (ExampleDef interface + lista)
│   └── units.ts                (forceUnit/dispUnit persistidos en localStorage)
├── zapata-aislada/             ← Ecuador NEC-SE-GC con selector tipo suelo
├── zapata-viga-amarre/         ← medianera + viga + centrada, Winkler 3D
├── plate-thin/, plate-thick/   ← Kirchhoff / Mindlin vía plateQ4Solve
├── membrana-pstress/           ← plane stress Q4 como muro de corte
├── shell-thin/, shell-thick/   ← Kirchhoff-Love / MITC4 vía deform+analyze
├── barra-axial/, truss-gen/    ← frames 1D
├── portico-2d/, tower-3d/, galpon/
├── edificio-aporticado/, edif-acero/, mezanine/
└── beams/                      ← Paz 6.3 (validación modal, ver abajo)
```

Todos los ejemplos se abren desde el workspace: `/workspace/?t=zapata-aislada`.
La dirección de antes, `/zapata-aislada/`, redirige ahí.

### Interfaz `ExampleDef` (examples/src/workspace/exampleRegistry.ts)

```ts
interface ExampleDef {
  id: string;
  name: string;
  category: string;
  params: Record<string, ParamDef>;
  build: (params, states, modalPanel?) => void;

  hasModal?: boolean;
  runModal?: (params, states, modalPanel) => void;

  onParamChange?: (changedKey, params) => void;   // soilType→q_adm, ks_factor, etc.
  computedLabels?: (params, states) => Record<string, string>;   // folder "📊 Calculados"
  inlineComputed?: Array<{ after, label, compute }>;             // ks debajo de ks_factor, etc.

  defaultShellResult?: string;        // "pressure" | "bendingXX" | ...
  availableShellResults?: string[];   // filtra dropdown Shell results
}

interface ParamDef {
  default: number;
  min?, max?, step?: number;
  label?: string;
  options?: Record<string, number>;   // dropdown numeric enum
  boolean?: boolean;                  // checkbox on/off (0/1)
  folder?: string;                    // agrupa en folder Tweakpane
}
```

### Flujo del workspace (examples/src/workspace/main.ts)

1. Usuario selecciona ejemplo → `loadExample(ex)`
2. `resetStates()` limpia objects3D, nodes, elements, inputs, outputs
3. `activeExampleVersion.v++` invalida van.derive de ejemplos previos (zapatas con springs reactivos)
4. `ex.build(params, states, modalPanel)` construye el modelo
5. Aplica `ex.defaultShellResult` al viewer y enciende loads/supports
6. `filterShellResultOptions(ex.availableShellResults)` filtra el dropdown
7. `autoScaleDeformedShape()` ajusta `deformScale` para max ≈ 25% del diagonal
8. `autoFitCamera()` reencuadra isométrica sobre el modelo
9. `buildParamsPane()` reconstruye Tweakpane con Vista / Unidades / Parámetros / Modal

## Solver FEM (hekatan-fem)

**API pública** (`hekatan-fem/src/index.ts`):
```ts
import { deform, analyze, modalAnalysis, modalAnalysisPaz,
         plateQ4Solve, didacticSolveCpp, slopeSRM } from "hekatan-fem";
```

| Función | Entrada | Salida |
|---|---|---|
| `deform` | nodes, elements, nodeInputs, elementInputs, springsList? | `DeformOutputs { deformations, reactions }` |
| `analyze` | nodes, elements, elementInputs, deformOutputs | `AnalyzeOutputs { bendingXX/YY/XY, membrane*, vonMises, pressure?, colorMapRanges? }` |
| `modalAnalysis` | nodes, elements, nodeInputs, elementInputs, nModes | `ModalOutputs { frequencies, modeShapes, massParticipation }` |
| `plateQ4Solve` | {E, nu, t, theoryType, mesh*, bcType, pressure} | `{ nodeResults[{w,bx,by}], elementResults[{Mxx,Myy,Mxy,Qx,Qy}] }` |

### Winkler springs nativos (extensión al `deform.cpp` original)

`deform()` acepta un 5º argumento opcional `springsList: Array<{node, dof, k}>`. En C++ ensambla:
```cpp
K_global.coeffRef(gdof, gdof) += k;   // gdof = 6*node + dof
```
Usado en zapata-aislada y zapata-viga-amarre para Winkler nodal (`k = ks × A_tributaria`).

### Placa Q4 con `theoryType`

```ts
plateQ4Solve({ theoryType: 0 })   // Mindlin-Reissner (FSDT, thick)
plateQ4Solve({ theoryType: 1 })   // Kirchhoff (CPT, thin)
plateQ4Solve({ theoryType: 2 })   // Plane stress (membrana)
```

**GDL de nudo, mano derecha** (3-sep-2026): `pointLoads`/`bcs`/`springs` con `dof` 0 = w,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GiorgioBurbanelli89/hekatan-struct-lineal](https://github.com/GiorgioBurbanelli89/hekatan-struct-lineal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
