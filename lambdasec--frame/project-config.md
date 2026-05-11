---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Frame is a separation logic entailment checker that uses Z3 SMT solver to verify entailments of the form `P |- Q`. The system supports:
- Core separation logic: empty heap (`emp`), points-to (`x |-> y`), separating conjunction (`*`)
- Pure formulas: equality, boolean logic, arithmetic
- Inductive predicates: lists, trees, custom data structures
- Frame reasoning with automatic inference

## Commands

### Testing
```bash
# Run all tests (553 tests: 532 pytest + 21 legacy suites)
python -m pytest tests/

# Run with verbose output
python -m pytest tests/ -v

# Run with coverage report
python -m pytest tests/ --cov=frame --cov-report=term

# Run specific test file
python -m pytest tests/test_footprint_analysis.py
python -m pytest tests/test_heap_graph_analysis.py

# Run specific legacy suite (all SL-COMP tests, old test files)
python -m pytest tests/test_legacy_suites.py -v

# Run tests matching a pattern
python -m pytest tests/ -k "footprint"
python -m pytest tests/ -k "slcomp"

# Run quietly (suppress warnings)
python -m pytest tests/ -q -W ignore::pytest.PytestCollectionWarning
```

### Benchmarking
```bash
# Run SL-COMP benchmarks
python benchmarks/run_slcomp.py

# Analyze failures
python benchmarks/analyze_failures.py

# Visualize heap structures
python benchmarks/visualize_heap.py
```

### Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run specific test cases interactively
python -c "from frame import EntailmentChecker; checker = EntailmentChecker(); print(checker.check_entailment('x |-> 5 * y |-> 3 |- x |-> 5'))"
```

## Architecture

The codebase is organized into logical modules for better maintainability:

```
frame/
├── core/           # Core abstractions (AST, parser)
├── encoding/       # Z3 SMT encoding
├── checking/       # Entailment checking and heuristics
├── analysis/       # Formula analysis and reasoning
├── heap/           # Heap graph and pattern detection
├── folding/        # Predicate folding/unfolding
├── arithmetic/     # Arithmetic reasoning
├── preprocessing/  # Formula preprocessing
├── predicates/     # Inductive predicate definitions
├── lemmas/         # Lemma library
└── utils/          # Utilities and proof management
```

### Core Components

**frame/core/ast.py** - Abstract Syntax Tree
- Defines all formula types: `Expr`, `Formula`, `Var`, `Const`, `ArithExpr`
- Spatial formulas: `Emp`, `PointsTo`, `SepConj`, `Wand`
- Pure formulas: `And`, `Or`, `Not`, `Eq`, `Neq`, `True_`, `False_`
- Quantifiers: `Exists`, `Forall`
- Predicate calls: `PredicateCall`

**frame/core/parser.py** - Parser
- Converts string formulas to AST
- Two-stage: Lexer tokenizes, Parser builds AST
- Supports entailment syntax: `"P |- Q"` for parsing both sides
- Handles precedence: quantifiers > disjunction > conjunction > separating conjunction

**frame/encoding/encoder.py** - Z3 Encoding
- Encodes separation logic formulas to Z3 SMT constraints
- Heap represented as Z3 array from locations to values
- Explicit domain tracking for allocated locations
- Key methods:
  - `encode_expr()`: Converts AST expressions to Z3
  - `encode_heap_assertion()`: Encodes spatial formulas with disjointness
  - `encode_pure()`: Encodes pure boolean/arithmetic constraints
- Delegates spatial encoding to `frame/encoding/_spatial.py`

**frame/checking/checker.py** - Entailment Checker
- Main interface: `EntailmentChecker` class
- Algorithm:
  1. Parse formulas to AST
  2. Unfold predicates (bounded, depth 3 default)
  3. Encode to Z3 with heap + domain tracking
  4. Check validity: `P |- Q` is valid if `P & !Q` is unsat
- Fast paths:
  - Reflexivity check (syntactic equality)
  - Heuristic checks before Z3 (via `frame/checking/heuristics.py`)
- Adaptive unfolding based on formula complexity
- Returns `EntailmentResult` with validity, model, and reason

**frame/predicates/** - Inductive Predicates
- `base.py`: `InductivePredicate` abstract base class
- `registry.py`: `PredicateRegistry` manages available predicates
- Built-in predicates:
  - `ListSegment`: `ls(x, y)` - list segment from x to y
  - `LinkedList`: `list(x)` - null-terminated list
  - `Tree`: `tree(x)` - binary tree
  - `DoublyLinkedList`: `dll(x, p, y, n)` - doubly-linked list
- `PredicateValidator`: Checks soundness (strict positivity, free variables, arity)
- `GenericPredicate`: Creates predicates from SMT2 definitions

**frame/lemmas/** - Lemma Library
- `base.py`: Core lemma library with pattern matching and application
- `_matcher.py`: Pattern matching for lemma application (meta-variables)
- `_substitution.py`: Substitution and equality normalization
- Proven lemmas for common predicates (e.g., list segment transitivity)
- Used to prove entailments without unfolding
- Key lemmas:
  - `ls_transitivity`: `ls(x,y) * ls(y,z) |- ls(x,z)`
  - `ls_cons`: `x |-> y * ls(y,z) |- ls(x,z)`
  - `ls_empty`: `ls(x,x) |- emp`

### Data Flow

1. **String Input** → Parser → AST
2. **AST** → Predicate Unfolding (recursive, bounded depth)
3. **Unfolded AST** → Z3 Encoder → SMT Constraints
4. **SMT Constraints** → Z3 Solver → SAT/UNSAT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lambdasec/frame](https://github.com/lambdasec/frame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
