---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bill's Lua Obfuscator is a **production-ready** web-based Lua code obfuscation tool that protects Lua source code by transforming it into functionally equivalent but harder-to-read code. The application runs entirely in the browser using Next.js 15+ and React 19+.

**Live Features:**

- Real-time Lua code obfuscation in the browser
- 10+ obfuscation techniques including advanced encryption and anti-debugging
- Beautiful, responsive UI that works on mobile, tablet, and desktop
- Monaco code editor with Lua syntax highlighting
- Real-time metrics display with detailed statistics
- Comprehensive test coverage (446 unit tests, 1,194 E2E tests across 12 test files)

## Architecture

### Implementation Status: ✅ **COMPLETE**

### Core Components

**Parser/Lexer Layer** (`lib/parser.ts`)

- Uses `luaparse` library to tokenize Lua source into AST
- Supports Lua 5.1, 5.2, 5.3 syntax
- Validates code and provides meaningful error messages
- Exports: `parseLua()`, `validateLua()`

**Transformation Engine** (`lib/obfuscator.ts`, `lib/obfuscator-simple.ts`)

- Applies multiple obfuscation techniques to the AST:
  - ✅ **Variable/function name mangling** - Hexadecimal identifier replacement
  - ✅ **String encoding** - Byte array transformation using string.char()
  - ✅ **Number encoding** - Mathematical expression obfuscation
  - ✅ **Control flow obfuscation** - Opaque predicates for complexity
  - ✅ **Code minification** - Whitespace and comment removal
- Configurable protection levels (0-100%)
- Preserves Lua standard library globals (print, pairs, math._, string._, etc.)
- Exports: `obfuscateLua()` with `ObfuscationOptions`

**Advanced Encryption Module** (`lib/encryption.ts`) - v1.1

- Multiple encryption algorithms for string obfuscation:
  - ✅ **XOR Cipher** - Rotating key encryption with position-dependent keys
  - ✅ **Base64 Encoding** - Custom alphabet base64 with scrambling
  - ✅ **Huffman Compression** - Frequency-based dictionary encoding
  - ✅ **Chunked Strings** - Multi-variable string distribution
- Generates Lua decryption functions inline
- Exports: `encryptString()`, `generateDecryptorCode()`

**Dead Code Injection Module** (`lib/dead-code.ts`) - v1.1

- Generates syntactically valid but semantically meaningless code:
  - ✅ **Unreachable blocks** - False conditionals with realistic code
  - ✅ **Dummy functions** - Unused but plausible-looking functions
  - ✅ **Variable pollution** - Meaningless variable declarations
  - ✅ **Fake operations** - Loops, table manipulations, math operations
- Configurable injection rate based on protection level
- Exports: `injectDeadCode()`, `generateDeadCode()`

**Control Flow Flattening Module** (`lib/control-flow.ts`) - v1.1

- Transforms linear code into state machines:
  - ✅ **State-based execution** - Convert sequential code to state dispatch
  - ✅ **Jump table obfuscation** - Indirect control flow
  - ✅ **Loop unrolling** - Expand small loops
  - ✅ **Condition splitting** - Break complex conditions
- Configurable intensity (0-100%)
- Exports: `applyControlFlowFlattening()`, `convertToStateMachine()`

**Anti-Debugging Module** (`lib/anti-debug.ts`) - v1.1

- Runtime checks to detect debugging attempts:
  - ✅ **Debug library detection** - Check for debug table
  - ✅ **Execution timing checks** - Detect debugger slowdown
  - ✅ **Stack depth validation** - Unusual call stack detection
  - ✅ **Integrity checks** - Code modification detection
  - ✅ **Environment validation** - Check for suspicious globals
- Configurable check types and frequency
- Exports: `generateAntiDebugFunction()`, `injectAntiDebugChecks()`

**Formatting Module** (`lib/formatter.ts`) - v1.1

- Configurable output code formatting:
  - ✅ **Minified** - Compact, no whitespace
  - ✅ **Pretty** - Readable with proper indentation
  - ✅ **Obfuscated** - Random spacing and line breaks
  - ✅ **Single-line** - Everything on one line
- Custom indent size and character options
- Exports: `formatCode()`, `addBlankLinesBetweenFunctions()`

**Metrics Module** (`lib/metrics.ts`) - v1.1

- Comprehensive obfuscation statistics:
  - ✅ **Size tracking** - Input/output bytes and lines
  - ✅ **Transformation counts** - Names, strings, numbers, dead code, anti-debug
  - ✅ **Performance metrics** - Duration, size ratios
  - ✅ **Processing mode** - Client-side indicator
- Real-time metrics calculation
- Exports: `calculateMetrics()`, `MetricsTracker`, `formatMetrics()`

**Code Generator** (`lib/generator.ts`)

- Converts transformed AST back to valid Lua source
- Handles 20+ Lua node types (functions, tables, loops, expressions, etc.)
- Maintains functional equivalence with original code
- Supports minification mode
- Exports: `generateCode()`

### Key Design Considerations

**Lua Version Compatibility**

- Different Lua versions have syntax/semantic differences
- 5.1: Most common in game modding (WoW, FiveM, etc.)
- 5.2+: Added goto statements, different \_ENV handling
- Consider target runtime environment when implementing transformations

**Performance vs Security Trade-offs**

- Heavy obfuscation increases execution time

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BillChirico/LUA-Obfuscator](https://github.com/BillChirico/LUA-Obfuscator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
