---
trigger: always_on
description: mathparse is a **mathematical expression evaluator** that safely parses and evaluates mathematical expressions from strings, supporting both numeric operators and natural language words across multiple languages.
---

# AI Agent Instructions for mathparse

## Architecture Overview

mathparse is a **mathematical expression evaluator** that safely parses and evaluates mathematical expressions from strings, supporting both numeric operators and natural language words across multiple languages.

### Core Components

- **`mathparse/mathparse.py`**: Main evaluation engine with three-phase pipeline
- **`mathparse/mathwords.py`**: Language definitions and multilingual mathematical vocabularies
- **`mathparse/__init__.py`**: Simple module entry point

### Key Data Flow

1. **Tokenization** (`tokenize()`) - Convert strings to mathematical tokens
2. **Word Replacement** (`replace_word_tokens()`) - Transform natural language to symbolic math
3. **Postfix Conversion** (`to_postfix()`) - Convert infix to postfix notation for safe evaluation
4. **Evaluation** (`evaluate_postfix()`) - Calculate result without using Python's `eval()`

**CRITICAL**: The postfix (RPN) evaluation system is the architectural foundation. Prefer extending the postfix evaluator over adding complex regex parsing when implementing new features.

## Critical Design Principles

### Security First
- **Never use `eval()`** - This is the project's core security principle
- All mathematical evaluation goes through controlled postfix evaluation
- Division by zero returns `'undefined'` string, never raises exceptions

### Postfix (RPN) Architecture Priority
- **Always prefer postfix evaluation over regex parsing** for new mathematical operations
- The `to_postfix()` and `evaluate_postfix()` functions handle operator precedence correctly
- Add new operators to `BINARY_OPERATORS` or `UNARY_FUNCTIONS` rather than complex regex rules
- Postfix notation eliminates ambiguity and provides consistent evaluation order
- When implementing new features, consider: "Can this be solved by extending the postfix evaluator?"
- **Think creatively about what constitutes an "operator"**: Binary operators don't just perform arithmetic—they can also construct values (e.g., decimal point combines integer and fractional parts)
- If a feature takes two inputs and produces one output, it can likely be modeled as a binary operator
- Before adding preprocessing logic, ask: "Could this be an operator with custom evaluation logic?"

### Multi-language Support
- Language codes follow **ISO 639-2** standard (3-letter codes like 'ENG', 'FRE')
- Each language in `MATH_WORDS` dict has structured sections:
  - `numbers`: word-to-number mappings
  - `binary_operators`: mathematical operations
  - `scales`: multipliers (hundred, thousand, etc.)
  - `prefix_unary_operators`: functions like "square root of"
  - `postfix_unary_operators`: suffixes like "squared"

### Generic Solutions Over Language-Specific
- **Prefer generic, cross-language solutions** over language-specific implementations
- When solving parsing issues, first consider if the solution can work for all languages
- Language-specific code should only be used when truly necessary for linguistic differences
- Examples of preferred approaches:
  - ✅ Extend `UNARY_FUNCTIONS` for new mathematical operations (works for all languages)
  - ✅ Improve operator precedence handling in postfix evaluation (universal benefit)
  - ✅ Enhance compound operator processing by length-based sorting (helps all languages)
  - ❌ Hard-code regex patterns for specific language character sets
  - ❌ Add language-specific preprocessing unless absolutely required
- If language-specific code is necessary, isolate it clearly and document why the generic approach wasn't sufficient

### Use mathwords.py Definitions, Avoid Hard-Coding
- **Always reference `mathwords.py` definitions** instead of hard-coding mathematical tokens
- Use `BINARY_OPERATORS`, `UNARY_FUNCTIONS`, and language dictionaries as the source of truth
- Examples of preferred approaches:
  - ✅ Use `mathwords.BINARY_OPERATORS` to get valid operators: `{'^', '*', '/', '+', '-'}`
  - ✅ Reference `words['binary_operators'].keys()` for language-specific operators
  - ✅ Check `mathwords.UNARY_FUNCTIONS.keys()` for valid unary functions
  - ✅ Use `mathwords.CONSTANTS` for mathematical constants
  - ✅ Create local copies for temporary modifications: `local_ops = mathwords.BINARY_OPERATORS.copy(); local_ops.add('(')`
  - ✅ Use set operations for combining: `mathwords.BINARY_OPERATORS | {'(', ')'}`
  - ❌ Hard-code operator strings like `"+-*/"` or `['+', '-', '*', '/']` in regex patterns
  - ❌ Hard-code function names like `['sqrt', 'log']` instead of using `UNARY_FUNCTIONS`
  - ❌ Duplicate mathematical definitions that already exist in `mathwords.py`
  - ❌ Directly mutate mathwords definitions: `ops = mathwords.BINARY_OPERATORS; ops.add('(')` (creates reference, modifies global state)
- **Key distinction**: Assignment creates a reference, not a copy. Use `.copy()` or set operations for local modifications
- **Benefits**: Ensures consistency, maintains single source of truth, automatically supports new operators when added
- **Pattern**: Import from `mathwords` and use the dictionaries: `from . import mathwords`

## Development Patterns

### Operator Design Philosophy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gunthercox/mathparse](https://github.com/gunthercox/mathparse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
