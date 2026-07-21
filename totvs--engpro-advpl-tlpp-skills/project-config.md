---
trigger: always_on
description: This workspace contains source code for **TOTVS Protheus ERP** in the **AdvPL** (`.prw`, `.prg`, `.prx`) and **TLPP** (`.tlpp`) languages, as well as automated tests in **Python** (TIR - Interface and e2e tests).
---

# AdvPL/TLPP Development Guidelines

## Language and Ecosystem

This workspace contains source code for **TOTVS Protheus ERP** in the **AdvPL** (`.prw`, `.prg`, `.prx`) and **TLPP** (`.tlpp`) languages, as well as automated tests in **Python** (TIR - Interface and e2e tests).

> **Default language**: All interactions, documentation, comments, commit messages, and reviews must be in **Brazilian Portuguese**, except for code identifiers (variable names, functions, classes) that follow technical conventions in English/Protheus abbreviations.

---

## Project Structure

```
Fontes_Doc/Master/Fontes/   → Main source code (~60+ business modules)
  ├── <Module>/              → E.g.: Financeiro/, Compras/, Faturamento/
  │   └── *.prw, *.tlpp     → Module sources
  ├── *.PRJ                  → Build manifests (source lists per module)
  └── Rdmake Padrao/         → RDMake compiler
Testes/Automação Protheus/<Country>/
  ├── <MODULE>/Scripts Web/  → TIR tests (Python) — end-to-end via Webapp
  └── <MODULE>/Dados/        → Test data
.agents/skills/   → AI agent skills for code generation, refactoring, documentation, tests, etc.
```

---

## Code Conventions

### Naming

- **Mandatory Hungarian notation**: `c` (character), `n` (numeric), `l` (logical), `a` (array), `o` (object), `d` (date), `b` (codeblock), `x` (variant), `j` (json)
- **Source file names**: Module prefix (4 chars) + number (3 digits) — e.g.: `MATA010`, `FINA138`, `ATFA002`
- **Table fields**: Table prefix (2-3 chars) + `_` + name — e.g.: `A1_COD`, `E1_FILIAL`
- **Tables**: Alias (2-3 chars) — e.g.: `SA1` (customers), `SE1` (accounts receivable), `SD1` (purchase invoice items)
- **Multilingual constants**: `STR0001` to `STR9999` defined in `.ch`

### Type System

#### AdvPL Types (optional static typing — type annotations with `as`)

| Type | Indicator | Description |
|------|-----------|-------------|
| Character (C) | `C` | Character strings from 0 to 65,535 characters (64 KB). Delimiters: `" "` or `' '` |
| Memo (M) | `M` | Equivalent to Character but with no defined size. Stored in data files (SYP) |
| Numeric (N) | `N` | Floating-point numeric values (integer and fractional). Guaranteed precision of 15 digits |
| Logical (L) | `L` | Logical values: `.T.` or `.Y.` (true), `.F.` or `.N.` (false) |
| Date (D) | `D` | Dates stored internally as Julian date |
| Fixed Size Decimal (F) | `F` | Fixed-size decimals for high precision |
| Array (A) | `A` | N-dimensional array of values |
| Code Block (B) | `B` | Executable code block that can be stored in variables |
| Undefined (U) | `U` | Undefined type (NIL) |
| Object (O) | `O` | Class objects |

#### TLPP Types (static typing — type annotations with `as`)

| Type | Indicator | Default Value | Description |
|------|-----------|---------------|-------------|
| `numeric` | `N` | `0` | Floating-point numeric values, positive or negative. Default numeric type |
| `integer` | `N` | `0` | Integer numeric values, positive or negative. Ideal for counters and loops |
| `double` | `N` | `0` | Floating-point numeric values, positive or negative |
| `decimal` | `F` | `Nil` | High-precision numeric values, essential for monetary calculations |
| `character` | `C` | `""` | Text values (alphanumeric, punctuation, special characters) |
| `logical` | `L` | `.F.` | Logical values: true (`.T.`) or false (`.F.`) |
| `date` | `D` | `31/12/1899` | Date storage |
| `array` | `A` | `Nil` | N-dimensional array of values |
| `object` | `O` | `Nil` | Interface or class objects |
| `json` | `J` | `Nil` | JSON objects |
| `codeblock` | `B` | `Nil` | Executable code blocks |
| `variant` | `U` | `Nil` | Variant and self-polymorphic type, can assume any available type |
| `variadic` | `J` | `-` | Used in function declarations for variable-length parameters (cannot instantiate variable) |

#### Key Differences

| Aspect | AdvPL | TLPP |
|--------|-------|------|
| Typing | Dynamic or optional static (not strongly typed) | Static (strongly typed) |
| Numeric Types | Only `numeric` | `numeric`, `integer`, `double`, `decimal` |
| Special Types | — | `json`, `variant`, `variadic` |
| Initialization | `Nil` by default | Automatic default values |
| Nil Comparison | Allowed | Restricted (`numeric`, `character`, `date`) |

#### Variable Declaration Syntax (TLPP)

```tlpp
// CORRECT: assignment BEFORE type annotation
Local cName   := "" as Character
Local nTotal  := 0  as Numeric
Local lFound  := .F. as Logical
Local aItems  := {} as Array
Local oModel  as Object          // without assignment is also valid

// WRONG: type annotation BEFORE assignment
Local cName as Character := ""   // does not compile

// WRONG: invalid type names
Local cName as String            // use Character
Local nVal  as Number            // use Numeric
Local lOk   as Boolean           // use Logical
```

Every function or method **must** follow this order, without exception:

```advpl
User Function MyFunction()     // 1. header
    Local  cVar := ""          // 2. Local  variables (all here)
    Private cPriv := ""        // 3. Private variables (all here)
    // executable code          // 4. logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [totvs/engpro-advpl-tlpp-skills](https://github.com/totvs/engpro-advpl-tlpp-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
