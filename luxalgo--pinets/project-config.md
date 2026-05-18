---
trigger: always_on
description: **PineTS** is a JavaScript/TypeScript library that enables the execution of Pine Script indicators in a JavaScript environment. It consists of two main components:
---

# PineTS - AI Agent Instructions

## Project Overview

**PineTS** is a JavaScript/TypeScript library that enables the execution of Pine Script indicators in a JavaScript environment. It consists of two main components:

1. **Pine Script Transpiler**: Converts native Pine Script v5+ code to PineTS syntax
2. **PineTS Runtime Transpiler**: Transforms PineTS syntax into executable JavaScript with proper time-series semantics

### Key Characteristics

-   **Dual Input Support**: Accepts both native Pine Script v5+ and PineTS syntax
-   **Runtime Transpilation**: Transforms code at runtime without requiring pre-compilation
-   **Pine Script v5+ Compatibility**: Full syntax support for TradingView's Pine Script
-   **Time-Series Processing**: Handles historical data with proper lookback capabilities
-   **Stateful Calculations**: Supports incremental technical analysis calculations
-   **Series-Based Architecture**: Everything is a time-series with forward storage and reverse access

## Architecture Documentation

Before making changes, familiarize yourself with the architecture:

-   **[Architecture Guide](docs/architecture/index.md)**: Main architecture overview
-   **[Transpiler](docs/architecture/transpiler/index.md)**: AST parsing, scope analysis, code transformation
    -   [Scope Manager](docs/architecture/transpiler/scope-manager.md): Variable renaming and unique ID generation
    -   [Transformers](docs/architecture/transpiler/transformers.md): AST transformation logic
    -   [Real Examples](docs/architecture/transpiler/examples.md): Actual transpilation output
-   **[Runtime](docs/architecture/runtime/index.md)**: Context, Series, and execution loop
    -   [Context Class](docs/architecture/runtime/context.md): The global state object
    -   [Series Class](docs/architecture/runtime/series.md): Forward storage with reverse access
    -   [Execution Flow](docs/architecture/runtime/execution-flow.md): Run loop and pagination
-   **[Namespaces](docs/architecture/namespaces/index.md)**: Implementation of `ta`, `math`, `request`, etc.
    -   [Technical Analysis (ta)](docs/architecture/namespaces/ta.md)
    -   [Math (math)](docs/architecture/namespaces/math.md)
    -   [Array (array)](docs/architecture/namespaces/array.md)
    -   [Request (request)](docs/architecture/namespaces/request.md)
    -   [Input (input)](docs/architecture/namespaces/input.md)
-   **[Debugging Guide](docs/architecture/debugging.md)**: Practical debugging techniques
-   **[Best Practices](docs/architecture/best-practices.md)**: Common pitfalls and recommended patterns

## Critical Concepts

### 1. Input Types: Pine Script vs PineTS Syntax

**CRITICAL**: PineTS accepts TWO different input formats. Understanding the difference is essential.

#### Detection Logic

```
Input Source
    │
    ├─ Is Function? ──────────────────→ Convert to string, treat as PineTS
    │
    └─ Is String?
           │
           ├─ Has //@version=X marker?
           │       │
           │       ├─ X >= 5 ──────────→ Pine Script → pineToJS pipeline
           │       └─ X < 5 ───────────→ Error (unsupported)
           │
           └─ No version marker ───────→ PineTS syntax (use as-is)
```

#### Pine Script v5+ (Native TradingView Syntax)

Detected by the `//@version=5` (or higher) marker. Goes through the `pineToJS` pipeline first.

```pinescript
//@version=5
indicator("EMA Cross")
fast = ta.ema(close, 9)
slow = ta.ema(close, 21)
plot(fast, "Fast EMA")
plot(slow, "Slow EMA")
```

#### PineTS Syntax (JavaScript-like)

No version marker. Uses JavaScript syntax with the `$` context object.

```javascript
($) => {
    const { close } = $.data;
    const { ta, plot } = $.pine;

    const fast = ta.ema(close, 9);
    const slow = ta.ema(close, 21);
    plot(fast, 'Fast EMA');
    plot(slow, 'Slow EMA');

    return { fast, slow };
};
```

#### JavaScript Function (Direct)

Functions are converted to string and treated as PineTS syntax.

```javascript
pineTS.run(($) => {
    const { close } = $.data;
    const { ta } = $.pine;
    return ta.sma(close, 20);
});
```

### 2. Transpiler Pipeline

The transpiler operates in two stages depending on input type:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        STAGE 1: Pine Script → PineTS                    │
│                    (Only for Pine Script input)                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Pine Script Input          pineToJS Pipeline           PineTS Output   │
│  ──────────────────    ─────────────────────────    ─────────────────   │
│  //@version=5          │ Lexer (tokenize)       │                       │
│  indicator("Test")     │ Parser (build AST)     │    ($) => {           │
│  sma = ta.sma(close,20)│ CodeGen (emit JS)      │      const {close}... │
│  plot(sma)             └─────────────────────────┘      ...             │
│                                                       }                 │
└─────────────────────────────────────────────────────────────────────────┘

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuxAlgo/PineTS](https://github.com/LuxAlgo/PineTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
