---
trigger: always_on
description: sPyTial is a Python library for spatial visualization of structured data using declarative constraints. It compiles to the Spytial diagramming language to generate interactive HTML visualizations. The library enables developers to visualize Python objects (trees, graphs, nested structures) with minimal effort while providing advanced spatial annotation capabilities.
---

# sPyTial: Spatial Python Visualization Library

sPyTial is a Python library for spatial visualization of structured data using declarative constraints. It compiles to the Spytial diagramming language to generate interactive HTML visualizations. The library enables developers to visualize Python objects (trees, graphs, nested structures) with minimal effort while providing advanced spatial annotation capabilities.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap and Install (15-20 seconds)
```bash
# Install the package in development mode
python -m pip install -e .
```
**NEVER CANCEL:** This typically takes 15-20 seconds. Set timeout to 60+ seconds.

### Install Development Dependencies (30-45 seconds)
```bash
# Install testing and linting tools
python -m pip install pytest>=7.0.0 flake8>=6.0.0 black>=23.0.0
```
**NEVER CANCEL:** This typically takes 30-45 seconds. Set timeout to 90+ seconds.

### Build the Package (10-15 seconds)
```bash
# Build source and wheel distributions
python -m pip install build
python -m build --no-isolation
```
**NEVER CANCEL:** Build takes 10-15 seconds. Set timeout to 60+ seconds.

### Run Tests (<1 second)
```bash
# Run test suite with pytest
python -m pytest test/ -v

# Run individual test files directly
python test/test_group_selector.py
python test/test_object_annotations.py
```
**Note:** Test suite currently has 2 failing tests out of 14 total, but core functionality works correctly. Do not modify code to fix unrelated test failures.

### Code Quality and Linting (immediate)
```bash
# Check code style - expect 270+ violations in current codebase
python -m flake8 spytial/ --count --statistics

# Check code formatting - expect 5 files need reformatting
python -m black spytial/ --check

# Apply code formatting
python -m black spytial/
```
**ALWAYS run `python -m black spytial/` and `python -m flake8 spytial/` before committing changes** or CI will fail.

## Validation Scenarios

**CRITICAL**: After making any changes, always run through these complete validation scenarios to ensure functionality is preserved:

### 1. Basic Visualization Test
```python
import spytial

# Test basic data structure visualization
data = {
    'name': 'John',
    'age': 30,
    'hobbies': ['reading', 'cycling'],
    'address': {
        'street': '123 Main St',
        'city': 'Boston'
    }
}
result = spytial.diagram(data, method='file', auto_open=False)
print(f"Generated: {result}")

# Verify HTML file was created and contains valid content
import os
assert os.path.exists(result)
with open(result, 'r') as f:
    content = f.read()
    assert 'html' in content.lower() and len(content) > 1000
print("✓ Basic visualization works")
```

### 2. Class-Level Annotation Test
```python
import spytial

# Test class decorators with spatial annotations
@spytial.group(field='children', groupOn=0, addToGroup=1)
@spytial.orientation(selector='value', directions=['above'])
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

node = Node('root', [Node('child1'), Node('child2')])
result = spytial.diagram(node, method='file', auto_open=False)
print(f"Generated annotated class diagram: {result}")
print("✓ Class annotations work")
```

### 3. Object-Level Annotation Test
```python
import spytial

# Test object-level spatial annotations
my_list = [1, 2, 3, 4, 5]
spytial.annotate_orientation(my_list, selector='items', directions=['horizontal'])
result = spytial.diagram(my_list, method='file', auto_open=False)
print(f"Generated object annotation diagram: {result}")
print("✓ Object annotations work")
```

### 4. Provider System Test
```python
import spytial

# Test data provider and serialization system
builder = spytial.CnDDataInstanceBuilder()
data = {'test': 'data', 'nested': {'values': [1, 2, 3]}}
instance = builder.build_instance(data)
assert isinstance(instance, dict)
print("✓ Provider system works")
```

## Optional Dependencies

Install these for enhanced functionality in demos:

```bash
# For pandas integration demos
python -m pip install pandas numpy matplotlib seaborn

# For Z3 constraint solving demos  
python -m pip install z3-solver

# All optional dependencies
python -m pip install pandas numpy matplotlib seaborn z3-solver
```

## Key Projects and Code Structure

### Core Library (`/spytial/`)
- `__init__.py` - Main API exports and aliases
- `visualizer.py` - HTML diagram generation (`diagram()` function)
- `annotations.py` - Spatial constraint decorators and object annotations
- `provider_system.py` - Data serialization and instance building
- `evaluator.py` - Spytial-Core evaluation engine integration
- `*_template.html` - HTML templates for visualization output

### Tests (`/test/`)
- `test_object_annotations.py` - Object-level annotation tests (2 failing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidprasad/spytial](https://github.com/sidprasad/spytial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
