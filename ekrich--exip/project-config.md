---
trigger: always_on
description: This document provides guidelines for AI assistants (Claude, GPT, Copilot, etc.) contributing to the EXIP project.
---

# AI Agent Guidelines for EXIP

This document provides guidelines for AI assistants (Claude, GPT, Copilot, etc.) contributing to the EXIP project.

## Project Overview

**EXIP** (Embeddable EXI Processor in C) is a C library for parsing and serializing EXI (Efficient XML Interchange) streams. It's designed for embedded systems with emphasis on low memory footprint and portability.

**License**: Apache 2.0 (see [LICENSE.txt](LICENSE.txt))

## Core Principles

### 1. EXI-First Architecture

- ✅ **Use EXI-encoded formats** for all schema processing
- ❌ **Do NOT add XML parsers** - EXIP is EXI-only by design
- ✅ Schemas must be EXI-encoded with `Preserve.prefixes` enabled
- ✅ Use existing `TreeTable` infrastructure from `src/grammarGen/`

### 2. Schema Format Support

**Supported:**
- ✅ **XSD (XML Schema Definition)** - ONLY this format
- Must be EXI-encoded using external tools:
  - **EXIficient** (Java) - Recommended, actively maintained
  - **OpenEXI** (Java) - Mentioned in EXIP documentation

**Explicitly Out of Scope:**
- ❌ **DTD** (Document Type Definition) - Will never be supported
- ❌ **RELAX NG** (RNG/RNC) - Not needed, adds unnecessary complexity
- ❌ Plain XML parsing - Use EXI format only

### 3. Dual Mode Support

All features must support **BOTH** processing modes:

1. **Schema-informed Mode**
   - Uses XML Schema for optimized encoding
   - Typed data handlers (intData, boolData, etc.)
   - Better compression and performance

2. **Schemaless Mode**
   - No schema required
   - String-only data handlers
   - More flexible but less efficient

**Critical**: Never design features that only work in one mode. Both modes must be first-class citizens.

## Coding Standards

### File Headers

All new source files should include the standard EXIP header:

```c
/*==================================================================*\
|                EXIP - Embeddable EXI Processor in C                |
|--------------------------------------------------------------------|
|          Licensed under the Apache License, Version 2.0            |
|  The full license terms and conditions are located in LICENSE.txt  |
\===================================================================*/
```

**Note**: Some older files reference "BSD 3-Clause License" in headers. The project transitioned to Apache 2.0 for explicit patent grant protections. All new code should use the Apache 2.0 header above, as specified in LICENSE.txt.

### Naming Conventions

- **Tools**: Prefix with `exip` (e.g., `exipg`, `exipd`, `exipe`)
- **Functions**: camelCase (e.g., `initParser`, `parseNext`)
- **Types**: PascalCase (e.g., `EXIStream`, `Parser`)
- **Struct members**: snake_case (e.g., `app_data`, `has_options`, `version_number`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `EXIP_OK`, `INDEX_MAX`)
- **Macros**: UPPER_SNAKE_CASE (e.g., `SET_STRICT`, `WITH_COMPRESSION`)

### File Organization

```
include/          - Public API headers
src/
  ├── common/     - Utilities (memory, hash tables, dynamic arrays)
  ├── contentIO/  - Parser and serializer implementations
  ├── grammar/    - Grammar management
  ├── grammarGen/ - Schema to grammar conversion
  ├── streamIO/   - Low-level bit/byte operations
  └── stringTables/ - String table management
examples/         - Sample code
docs/user/        - Public user documentation (TeX format)
info/             - Internal project documentation (Markdown)
scripts/          - various script to run locally or via CI
```

### Error Handling

```c
errorCode tmp_err_code = EXIP_UNEXPECTED_ERROR;

// Propagate errors
TRY(functionCall());

// Handle errors before propagating
TRY_CATCH(functionCall(), {
    // Cleanup code
    freeAllocList(&list);
});
```

### Memory Management

- Use `AllocList` for tracked allocations
- Use `EXIP_MALLOC()` and `EXIP_FREE()` macros
- Clean up with `freeAllocList()` on error paths
- Dynamic arrays grow in chunks (configured per array)

### String Handling

- Strings are length-prefixed: `struct String { CharType* str; Index length; }`
- Default implementation is ASCII (can be replaced for Unicode)
- Use `asciiToString()` for conversions
- Never assume null-termination unless explicitly created

## Architecture Patterns

### 1. Content Handler Pattern (Observer)

```c
// Parser invokes callbacks
ContentHandler handler;
handler.startElement = my_startElement;
handler.stringData = my_stringData;
parser.handler = handler;
```

### 2. Function Pointer Tables

```c
// Serializer API
const EXISerializer serialize;
serialize.startDocument(&stream);
serialize.startElement(&stream, qname, &valueType);
```

### 3. Grammar State Machine

- States = Non-terminals (grammar rules)
- Transitions = Productions
- Input = Event codes

### 4. Tracked Memory Allocation

```c
AllocList memList;
initAllocList(&memList);
void* ptr = EXIP_MALLOC(&memList, size);
// ... use ptr ...
freeAllocList(&memList);  // Frees all tracked allocations
```

## Common Patterns

### Unified Data Handling (Schema + Schemaless)

```c
// Generic data value
typedef union {
    Integer intVal;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekrich/exip](https://github.com/ekrich/exip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
