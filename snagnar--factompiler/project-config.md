---
trigger: always_on
description: The compiler transforms Facto source code into Factorio blueprint strings through a 5-stage pipeline:
---

# General Agent Guidelines

# Compiler Architecture Overview

The compiler transforms Facto source code into Factorio blueprint strings through a 5-stage pipeline:

```
Source (.facto) → Parser → Semantic Analyzer → AST Lowerer → Layout Planner → Blueprint Emitter → Blueprint
```

## Directory Structure

```
dsl_compiler/
├── grammar/
│   └── facto.lark              # Lark grammar definition for Facto
├── src/
│   ├── ast/                    # AST node definitions
│   │   ├── base.py             # Base AST node class with source location
│   │   ├── expressions.py      # Expression nodes (binary ops, function calls, etc.)
│   │   ├── literals.py         # Literal nodes (numbers, signals, etc.)
│   │   └── statements.py       # Statement nodes (assignments, memory, etc.)
│   │
│   ├── parsing/                # Stage 1: Source → AST
│   │   ├── parser.py           # Main parser using Lark, entry: DSLParser.parse()
│   │   ├── preprocessor.py     # Import resolution and preprocessing
│   │   └── transformer.py      # Lark tree → AST node transformation
│   │
│   ├── semantic/               # Stage 2: Type checking & validation
│   │   ├── analyzer.py         # Main analyzer, entry: SemanticAnalyzer.visit()
│   │   ├── symbol_table.py     # Symbol table for variable tracking
│   │   └── type_system.py      # Type definitions and type checking logic
│   │
│   ├── lowering/               # Stage 3: AST → IR
│   │   ├── lowerer.py          # Main lowerer, entry: ASTLowerer.lower_program()
│   │   ├── expression_lowerer.py  # Expression to IR translation
│   │   ├── statement_lowerer.py   # Statement to IR translation
│   │   ├── memory_lowerer.py   # Memory operations to IR
│   │   └── constant_folder.py  # Compile-time constant evaluation
│   │
│   ├── ir/                     # Intermediate Representation
│   │   ├── nodes.py            # IR node definitions (IROperation, etc.)
│   │   ├── builder.py          # IRBuilder for constructing IR
│   │   └── optimizer.py        # CSE and constant propagation optimizers
│   │
│   ├── layout/                 # Stage 4: IR → Physical Layout
│   │   ├── planner.py          # Main planner, entry: LayoutPlanner.plan_layout()
│   │   ├── layout_plan.py      # LayoutPlan data structure
│   │   ├── entity_placer.py    # Entity placement algorithms
│   │   ├── connection_planner.py  # Wire connection planning
│   │   ├── memory_builder.py   # Memory cell layout construction
│   │   ├── wire_router.py      # Wire routing and MST optimization
│   │   ├── signal_analyzer.py  # Signal flow analysis
│   │   ├── signal_graph.py     # Signal dependency graph
│   │   ├── power_planner.py    # Power pole placement
│   │   └── tile_grid.py        # 2D grid management
│   │
│   ├── emission/               # Stage 5: Layout → Blueprint
│   │   ├── emitter.py          # Main emitter, entry: BlueprintEmitter.emit_from_plan()
│   │   └── entity_emitter.py   # Entity-specific emission logic
│   │
│   └── common/                 # Shared utilities
│       ├── constants.py        # Compiler configuration constants
│       ├── diagnostics.py      # Error/warning collection
│       ├── entity_data.py      # Factorio entity definitions
│       ├── signal_registry.py  # Signal type registry
│       ├── signals.py          # Signal utilities
│       └── source_location.py  # Source code location tracking
```

## Key Entry Points

- **compile.py** - CLI tool, orchestrates the full pipeline
- **DSLParser.parse(source, filename)** - Parse source to AST
- **SemanticAnalyzer.visit(ast)** - Type check and validate
- **ASTLowerer.lower_program(ast)** - Lower AST to IR
- **LayoutPlanner.plan_layout(ir_ops)** - Plan physical layout
- **BlueprintEmitter.emit_from_plan(plan)** - Emit blueprint

## Minimal Pipeline Example

```python
#!/usr/bin/env python3
"""Minimal example of running the compilation pipeline."""
from pathlib import Path
from dsl_compiler.src.parsing.parser import DSLParser
from dsl_compiler.src.semantic.analyzer import SemanticAnalyzer
from dsl_compiler.src.lowering.lowerer import ASTLowerer
from dsl_compiler.src.layout.planner import LayoutPlanner
from dsl_compiler.src.emission.emitter import BlueprintEmitter
from dsl_compiler.src.common.diagnostics import ProgramDiagnostics

def compile_source(source: str) -> str:
    """Compile Facto source code to blueprint string.""""
    diagnostics = ProgramDiagnostics()
    
    # Stage 1: Parse
    parser = DSLParser()
    ast = parser.parse(source, "<string>")
    
    # Stage 2: Semantic Analysis
    analyzer = SemanticAnalyzer(diagnostics=diagnostics)
    analyzer.visit(ast)
    
    # Stage 3: Lower to IR
    lowerer = ASTLowerer(analyzer, diagnostics)
    ir_ops = lowerer.lower_program(ast)
    
    # Stage 4: Plan Layout
    planner = LayoutPlanner(
        lowerer.ir_builder.signal_type_map,
        diagnostics=diagnostics,
        signal_refs=lowerer.signal_refs,
        referenced_signal_names=lowerer.referenced_signal_names,
    )
    layout = planner.plan_layout(ir_ops)
    
    # Stage 5: Emit Blueprint
    emitter = BlueprintEmitter(diagnostics, lowerer.ir_builder.signal_type_map)
    blueprint = emitter.emit_from_plan(layout)
    
    return blueprint.to_string()

# Example usage:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snagnar/Factompiler](https://github.com/Snagnar/Factompiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
