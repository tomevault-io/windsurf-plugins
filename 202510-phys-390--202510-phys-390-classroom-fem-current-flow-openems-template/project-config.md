---
trigger: always_on
description: - OpenEMS FDTD electromagnetic simulator
---

# OpenEMS + ElmerFEM Development Container - Project Summary

## Branch Structure

**main branch:**
- OpenEMS FDTD electromagnetic simulator
- High-frequency signal integrity analysis
- VTK field visualization for ParaView

**ElmerFEM branch:**
- Adds ElmerFEM finite element solver
- DC/low-frequency current flow analysis
- Power distribution network analysis
- Combines both simulation tools

## What's Working - OpenEMS (main + ElmerFEM branches)

### Docker Environment
- ✅ Dockerfile builds successfully (~20 min on Apple Silicon)
- ✅ All dependencies installed (OpenEMS, CSXCAD, VNC, etc.)
- ✅ x86_64 emulation working on ARM
- ✅ Image size: ~2-3 GB
- ✅ Single `setup_gui.sh` script starts everything

### VNC Desktop
- ✅ XFCE4 desktop starts correctly
- ✅ Accessible via http://localhost:6080/vnc.html
- ✅ Password: `openems`
- ✅ Matplotlib plots display in VNC window
- ✅ Ctrl+C gracefully shuts down

### OpenEMS Installation
- ✅ CSXCAD v0.6.3 compiled and installed
- ✅ OpenEMS v0.0.36 compiled and installed
- ✅ Python bindings working
- ✅ All imports successful (CSXCAD, openEMS, numpy, matplotlib, h5py)
- ✅ Official tutorials working perfectly

### OpenEMS Field Visualization - **WORKING!**
- ✅ **plane_wave_simple.py** - Clean electromagnetic field propagation
  - Plane wave excitation (no port configuration issues)
  - Shows E-field and H-field propagation
  - Demonstrates wave interaction with dielectric
  - Exports VTK files for ParaView visualization
  - ~1-2 minute simulation, ~2000 VTK files

- ✅ **simple_field_dump.py** - Minimal field visualization example

- ✅ **microstrip_with_vtk.py** - Advanced microstrip with fields
  - Has port configuration warnings but VTK dumps work independently
  - Shows fields around PCB trace

### Critical Fix: OpenEMS Coordinate System
**Problem discovered:** All geometry in OpenEMS (materials, excitations, dumps, ports) must use **mesh coordinates**, NOT SI units!

**Wrong:**
```python
mesh.SetDeltaUnit(1e-3)  # mm to meters
mesh.AddLine('x', np.linspace(0, 40, 21))  # 0 to 40 mm
dump.AddBox(start=[0, 0, 0], stop=[40*unit, 40*unit, 60*unit])  # ✗ SI units
```

**Correct:**
```python
mesh.SetDeltaUnit(1e-3)  # mm to meters
mesh.AddLine('x', np.linspace(0, 40, 21))  # 0 to 40 mm
dump.AddBox(start=[0, 0, 0], stop=[40, 40, 60])  # ✓ Mesh coordinates
```

This fix was applied to all geometry definitions: materials, excitations, dumps, and ports.

### ParaView Visualization
- ✅ VTK export working correctly
- ✅ 3D electromagnetic field data
- ✅ Time-evolution animation of wave propagation
- ✅ PARAVIEW_GUIDE.md created with complete instructions

## What's Working - ElmerFEM (ElmerFEM branch only)

### ElmerFEM Installation
- ✅ Built from source (~15-30 min additional build time)
- ✅ ElmerSolver, ElmerGrid, ElmerGUI installed
- ✅ MUMPS and Hypre solvers included
- ✅ Gmsh mesh generator (Python API)
- ✅ Python packages: gmsh, meshio, pygmsh, pyvista

### ElmerFEM Examples - **WORKING!**
- ✅ **simple_resistor.py** - Basic test (rectangle with voltage applied)
  - Validates ElmerFEM installation
  - Simple 2D conductor
  - Direct solver, guaranteed convergence

- ✅ **tapered_working.py** - Tapered PCB trace with current density
  - 2D copper trace: 4mm → 1mm width over 15mm length
  - DC current flow analysis
  - Shows current crowding in narrow section
  - Exports: Potential, Electric Field, Current Density, Joule Heating
  - VTK output for ParaView visualization

### Critical Fix: ElmerFEM Field Export
**Problem:** ElmerFEM calculates fields but doesn't automatically export them to VTU files.

**Solution:** Use `Exported Variable` in solver configuration:
```
Solver 1
  Calculate Electric Field = True
  Calculate Current Density = True

  ! Force export to VTU
  Exported Variable 1 = -dofs 3 Electric Field
  Exported Variable 2 = -dofs 3 Current Density
End
```

Also must use **Direct Solver (UMFPack)** instead of iterative solvers for complex geometries to ensure convergence.

### ElmerFEM vs OpenEMS

**Use ElmerFEM for:**
- DC or low-frequency (<1 MHz) analysis
- Resistive voltage drops (IR drop)
- Power distribution networks
- Current density in conductors
- Thermal-electrical coupling
- Static field problems

**Use OpenEMS for:**
- High-frequency (>10 MHz) analysis
- Transmission lines, S-parameters
- Signal integrity, impedance
- Antennas, RF circuits
- Wave propagation
- Time-domain electromagnetics

## Student Workflow

### Start GUI (both branches)
```bash
bash setup_gui.sh
# Opens: http://localhost:6080/vnc.html
# Password: openems
```

### OpenEMS Examples
```bash
cd /workspace/examples
python3 plane_wave_simple.py  # Clean field visualization, no port errors
python3 simple_field_dump.py   # Minimal example
```

### ElmerFEM Examples (ElmerFEM branch)
```bash
cd /workspace/elmerfem-examples
python3 simple_resistor.py     # Basic validation test
python3 tapered_working.py     # Tapered trace with current density
```

### Official OpenEMS Tutorials
```bash
cd /workspace/Tutorials
python3 Rect_Waveguide.py
# Plots appear in VNC desktop
```

## Documentation

### Main Documentation
- ✅ **README.md** - Quick start guide
- ✅ **PARAVIEW_GUIDE.md** - Complete ParaView visualization guide
- ✅ **Tutorials/README.md** - Official OpenEMS tutorials guide

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [202510-PHYS-390/202510-phys-390-classroom-fem-current-flow-openems-template](https://github.com/202510-PHYS-390/202510-phys-390-classroom-fem-current-flow-openems-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
