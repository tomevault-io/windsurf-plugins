---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GELSA (version 8.8) is a **Simulated Annealing-based analog IC layout generator** written in C. It performs placement and global routing of analog circuit cells, optimizing a multi-objective cost function that considers area, wirelength, symmetry constraints, wells, abutment, and other analog-specific layout requirements. Developed at Centro Nacional de Microelectronica, Sevilla (1992-2000).

## Build Commands

All build commands run from `code/` directory (the makefile is at `code/makefile`):

```bash
cd code

# Optimized build (-O2 -std=gnu89), outputs code/gelsa.exe
make rapido

# Debug build (-g), outputs code/gelsa.g
make xdbx

# Build graph generator tool
make generador

# Build AGA interface (ALSYN/OCTTOOLS)
make interface

# Build all targets
make all

# Clean
make clean
```

Compiler is `gcc`. Object files go to `code/objects/` (optimized) or `code/objects_g/` (debug). Links against `-lm`.

## Running

GELSA must be run from a directory that contains a `gelsa/` subdirectory with input files:

```bash
# Run from example_perfd3 which has gelsa/ with all input files
cd example_perfd3
../code/gelsa.exe

# With visual control
../code/gelsa.exe -visual

# With redimensioning
../code/gelsa.exe -redim
```

See `example_perfd3/gelsa/` for a complete set of input files. Variants `gelsa.ori/`, `gelsa.run1/`, `gelsa.run2/` contain saved runs with outputs for comparison.

## Input Files (in `gelsa/` directory)

| File | Description |
|------|-------------|
| `netcell.in` | Cell descriptions: names, dimensions, pin positions, net connectivity. Lines start with `N:` (NMOS), `P:` (PMOS), or `C:` (capacitor). Pins tagged as `D:` (drain), `G:` (gate), `S:` (source) |
| `grafo.in` | Initial slicing tree as a single line of cell IDs and `H`/`V` cut operators |
| `pars.in` | SA parameters: temperatures, cooling schedule (alfa, nitert, cond, tfin), cost weights (cnets, carea, csim, cwells, etc.), and feature flags (GR type, WELLS, ABUTMENT, etc.) |
| `pesos.in` | Net weights and types (PAD, SUPPLY) |
| `simloc.in` | Local symmetry group definitions (CENTROIDE, STACKED, MATCHING, EJE, ARRAY, etc.) |
| `simglob.in` | Global symmetry constraints (optional) |
| `rest_rou.in` | Analog routing restrictions |
| `max.in` | Pre-computed normalization parameters (NETS_MAX, RESTS_MAX, DENS_MAX) - optional, auto-computed if absent |

## Output Files (in `gelsa/` directory)

`grafo.out`, `netcell.out`, `final.out`, `coste.out`, `funcion.out`, `routing.out`, `canales.out`, `info.out`, `info2.out`, `cparc.out`, `conts.out`, `edif_gelsa.out`, `*.fig` (graphical), `gelsa2alsyn.conv`, `tree.redim`.

## Architecture

### Global State Pattern
All major data structures are **global variables** declared in `include/intro.h` (definitions) and `include/extern_intro.h` (extern declarations). Every source file except `main.c` includes `extern_intro.h`.

### Key Data Structures (`include/tipos.h`)
- **`t_ARBOL (TREE, TROLD, TRCACHOS)`** - Slicing tree representation: each node holds a cell ID or cut type (CH/CV), coordinates, dimensions, rotation/mirror properties, and symmetry links
- **`t_CELDA (CBAS[])`** - Base cell descriptions with pin info, dimensions, symmetry groups, cell type (NMOS/PMOS)
- **`t_NUDOS (NET[])`** - Electrical nets with bounding box, pin lists, and channel connectivity
- **`t_CANAL (CHAN[])`** - Routing channels between cells with breakpoint geometry
- **`t_HUECO (HOLE[])`** - Holes (gaps) between cells in the layout
- **`t_SIM (SIM[])`** - Symmetry constraint groups
- **`t_NESO / t_GEO / t_BREAKPOINT`** - Contour representation (N/E/S/O sides) with breakpoint geometry for pin/channel tracking
- **`t_control_shapes (SHCELDAS)`** - Cell shape variants (each cell can have multiple aspect ratios across 8 rotation/mirror transforms)

### Core Algorithm Flow (`main.c`)
1. Read cell descriptions with shapes (`lee_netcell_in_shapes`)
2. Read symmetry constraints (`lee_sim_in`, `lee_sim_globales`)
3. Read net weights and SA parameters (`lee_pesos_in`, `lee_datos_SA`)
4. Allocate graph structures for GR15 routing
5. Read initial slicing tree (`lee_grafo`)
6. Compute initial temperature (`tini()` in `enfria.c`) - evaluates 100+250 random moves
7. **Simulated Annealing loop** (`sal()` in `enfria.c`) - outer cooling loop with inner Metropolis acceptance
8. Output results (graphical, EDIF, ALSYN)

### Source Modules

| File | Role |
|------|------|
| `main.c` | Entry point, orchestration, summary output |
| `enfria.c` | SA engine: initial temperature (`tini`), cooling loop (`sal`), stop condition |
| `mueve.c` | Move generation (`gen_mov`): 11 operator types (R_C, C_O, I_C, I_CS, I_SG, R_SG, IN_SG, LL_H, M_C, M_SG, C_SH) with adaptive selection |
| `eval_costes.c` | Multi-objective cost function evaluation |
| `trata_arbol.c` | Slicing tree operations: dimensioning, positioning, contour computation |
| `lee_datos.c` | Input parsing for all `gelsa/*.in` files |
| `ini_estructuras.c` | Memory allocation and data structure initialization |
| `grafo.c` | Graph construction for routing (node/edge/ligature allocation) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juanpri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
