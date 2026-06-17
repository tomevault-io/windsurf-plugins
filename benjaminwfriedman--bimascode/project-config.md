---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BIM as Code (bimascode) is a Python library for programmatic Building Information Modeling. It enables creating buildings through code, generating documentation drawings, and exporting to IFC and DXF formats. Built on build123d (CAD geometry), IfcOpenShell (IFC), and ezdxf (DXF).

## Development Commands

### Setup
```bash
# Development installation with all dependencies
pip install -e ".[dev,viz]"
```

### Testing
```bash
# Run all tests with coverage
pytest

# Run specific test file
pytest tests/test_walls.py

# Run specific test class or function
pytest tests/test_walls.py::TestWall::test_wall_creation

# Run tests matching a pattern
pytest -k "wall_join"
```

### Code Quality
```bash
# Format code (100 char line length)
black src/ tests/

# Lint with ruff
ruff check src/ tests/

# Type check with mypy
mypy src/
```

### Documentation
```bash
# Regenerate API docs (output to docs/)
pdoc -o docs src/bimascode
```

### Running Examples
```bash
# Activate virtual environment first
source venv/bin/activate

# Run example scripts from project root
python examples/sprint6_demo.py
python examples/school_floor_plan.py
python examples/office_world_geometry_demo.py
```

**Note:** Always activate the virtual environment (`source venv/bin/activate`) before running tests, examples, or any Python commands.

## Architecture Overview

### Core Design Patterns

**Type/Instance Pattern**: All architectural and structural elements use a parametric type/instance system similar to Revit/BIM systems.

- `ElementType` (base class in `core/type_instance.py`): Defines shared parameters for all instances
  - Examples: `WallType`, `FloorType`, `DoorType`, `ColumnType`
  - When type parameters change, all instances are notified
  - Type holds reference to all its instances

- `ElementInstance` (base class in `core/type_instance.py`): Individual placed elements
  - Examples: `Wall`, `Floor`, `Door`, `Column`
  - Can override type parameters with instance-specific values
  - Geometry is cached and regenerated only when parameters change
  - All instances inherit from both `ElementInstance` and a geometry mixin

**World Geometry Mixins**: Two mixin classes handle transformation from local to world coordinates:

- `FreestandingElementMixin` (`core/world_geometry.py`): For elements positioned directly in world space
  - Used by: Wall, Floor, Ceiling, Column, Beam
  - Subclasses implement `_get_world_position()` and `_get_world_rotation()`

- `HostedElementMixin` (`core/world_geometry.py`): For elements positioned relative to a host
  - Used by: Door, Window
  - Subclasses implement `_get_host_transform()` and `_get_local_transform()`

**Expose Parameters to Driver**: All classes should provide sensible defaults for common settings, but every parameter must be configurable from driver files (example scripts, application code). Library internals should never hardcode values that users might need to customize.

```python
# GOOD - defaults with override capability
class Sheet:
    def __init__(
        self,
        size: SheetSize,
        number: str = "",
        margins: tuple[float, float, float, float] = (10.0, 10.0, 10.0, 10.0),  # default
    ):
        self._margins = margins

# BAD - hardcoded internal value
class Sheet:
    def __init__(self, size: SheetSize, number: str = ""):
        self._margins = (10.0, 10.0, 10.0, 10.0)  # no way to override
```

### Critical build123d Behaviors

**IMPORTANT**: build123d has non-intuitive transformation behavior that causes subtle bugs:

1. **`locate()` modifies geometry IN PLACE** - Always `copy.copy()` before transforming:
   ```python
   # WRONG - corrupts cached geometry
   return self.get_geometry().locate(transform)

   # CORRECT - copy first
   import copy
   geom_copy = copy.copy(self.get_geometry())
   return geom_copy.locate(transform)
   ```

2. **`locate()` REPLACES transforms, doesn't chain** - Compose transforms using multiplication:
   ```python
   # WRONG - second locate() loses first transform
   box.locate(local_transform)
   box.locate(world_transform)

   # CORRECT - compose with multiplication before locate()
   combined = world_transform * local_transform
   box.locate(combined)
   ```

3. **`Polygon()` auto-centers at centroid** - Created polygons shift vertices to center at origin

See `docs/build123d_behavior.md` for detailed explanations and examples.


### Module Organization

```
src/bimascode/
├── core/               # Base classes and patterns
│   ├── element.py      # Element base class with GUID, properties, cache
│   ├── type_instance.py # ElementType/ElementInstance parametric pattern
│   └── world_geometry.py # Freestanding/Hosted geometry transformation mixins
├── spatial/            # Project hierarchy
│   ├── building.py     # Building (IFC project root), unit system management
│   ├── level.py        # Building storeys with elevation tracking
│   ├── grid.py         # Layout axes (numeric/alphabetic)
│   └── room.py         # Spatial zones with area/volume
├── architecture/       # Architectural elements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benjaminwfriedman/bimascode](https://github.com/benjaminwfriedman/bimascode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
