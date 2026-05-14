---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

kicad-sch-api is a Python library for reading and writing KiCAD schematic files. The core focus is **exact format preservation** and **simple API design**.

**Key Principle**: "Simple is my north star" - prioritize simplicity over features.

## CRITICAL: KiCAD Coordinate System ⚠️

### 🔴 THE MOST IMPORTANT CONCEPT IN THIS LIBRARY

**KiCAD uses TWO DIFFERENT coordinate systems**, and understanding this is CRITICAL for ALL pin position calculations, connectivity analysis, and component placement.

### The Two Coordinate Systems

#### 1. Symbol Space (Library Definitions)
**Uses NORMAL Y-axis (+Y is UP)** like standard mathematics:
- All KiCAD symbol libraries define pins in this coordinate system
- Pin 1 at `(0, 3.81)` means 3.81mm UPWARD from origin
- Pin 2 at `(0, -3.81)` means 3.81mm DOWNWARD from origin

#### 2. Schematic Space (Placed Components)
**Uses INVERTED Y-axis (+Y is DOWN)** like most computer graphics:
- **Lower Y values** = visually HIGHER on screen (top)
- **Higher Y values** = visually LOWER on screen (bottom)
- **X-axis is normal** (increases to the right)

### The Critical Transformation

**When placing a symbol on a schematic, Y coordinates MUST be negated:**

```python
# Symbol library defines (NORMAL Y-axis, +Y up):
Pin 1: (0, +3.81)   # 3.81mm UPWARD in symbol space
Pin 2: (0, -3.81)   # 3.81mm DOWNWARD in symbol space

# Component placed at (100, 100) in schematic (INVERTED Y-axis, +Y down):
# Y must be NEGATED during transformation:
Pin 1: (100, 100 + (-3.81)) = (100, 96.52)   # VISUALLY AT TOP (lower Y)
Pin 2: (100, 100 + (+3.81)) = (100, 103.81)  # VISUALLY AT BOTTOM (higher Y)
```

### Implementation Location

**File:** `kicad_sch_api/core/geometry.py:apply_transformation()`

```python
def apply_transformation(point, origin, rotation, mirror):
    x, y = point

    # CRITICAL: Negate Y to convert from symbol space (normal Y) to schematic space (inverted Y)
    # This MUST happen BEFORE rotation/mirroring
    y = -y
    logger.debug(f"After Y-axis inversion (symbol→schematic): ({x}, {y})")

    # Then apply mirroring and rotation...
```

**This single line (`y = -y`) is CRITICAL** - without it, all pin positions are swapped and connectivity analysis breaks.

### Why This Matters - Real Impact

**Without this transformation, the library had a catastrophic bug:**
- ❌ All pin numbers were swapped (pin 1 calculated at bottom, pin 2 at top)
- ❌ All connectivity analysis was broken
- ❌ Wire routing connected to wrong pins
- ❌ Hierarchical connections failed
- ❌ Netlist generation was incorrect

**With this transformation:**
- ✅ Pin positions match KiCAD exactly
- ✅ Connectivity analysis works correctly
- ✅ Wire routing connects to correct pins
- ✅ Hierarchical connections work
- ✅ Netlist matches `kicad-cli` output

### Discovery Timeline

This was discovered during hierarchical connectivity implementation:
1. Initial implementation had pins swapped
2. User pointed out: "pin 2 is a higher number than pin 1, but pin 1 is at top"
3. Root cause: Symbol libraries use normal Y-axis, schematics use inverted Y-axis
4. Fix: Add Y negation BEFORE rotation/mirroring
5. Result: All 11 hierarchical connectivity tests pass + all pin rotation tests pass

### Testing Coverage

**Tested by:**
- `tests/unit/test_pin_rotation.py` - 11 tests for pin positions at all rotations
- `tests/reference_tests/test_pin_rotation_*.py` - 8 reference tests against real KiCAD schematics
- `tests/unit/test_connectivity_ps2_hierarchical.py` - 11 tests for hierarchical connections
- All tests verify pin positions match `kicad-cli` netlist output

### When Working with Pin Positions

**ALWAYS remember:**
- Pin positions from `list_component_pins()` are in SCHEMATIC space (inverted Y)
- Lower Y = visually HIGHER on screen (top of component)
- Higher Y = visually LOWER on screen (bottom of component)

**Example with resistor at position (100, 100) with rotation=0:**
```python
# Pin positions returned by list_component_pins():
Pin 1: (100, 96.52)   # Lower Y = VISUALLY AT TOP ⬆️
Pin 2: (100, 103.81)  # Higher Y = VISUALLY AT BOTTOM ⬇️
```

### Grid Alignment

**ALL positions in KiCAD schematics MUST be grid-aligned:**

- **Default grid:** 1.27mm (50 mil) - the standard KiCAD schematic grid
- **Component positions** must be on grid
- **Wire endpoints** must be on grid
- **Pin positions** must be on grid
- **Label positions** should be on grid
- **Sheet boundaries** should be on grid-aligned rectangles
- **Junction positions** must be on grid

**Common grid-aligned values (in mm):**
```
0.00, 1.27, 2.54, 3.81, 5.08, 6.35, 7.62, 8.89, 10.16, ...
```

**When creating components programmatically:**
- Always use grid-aligned coordinates
- Round positions to nearest 1.27mm increment
- Verify pin positions align to grid after transformations

**Example - Grid-aligned component placement:**
```python
# Good - on grid
sch.components.add('Device:R', 'R1', '10k', position=(100.33, 101.60))

# Bad - off grid
sch.components.add('Device:R', 'R2', '10k', position=(100.5, 101.3))
```

**Why grid alignment matters:**
- Ensures proper electrical connectivity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [circuit-synth/kicad-sch-api](https://github.com/circuit-synth/kicad-sch-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
