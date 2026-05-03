---
trigger: always_on
description: This VS Code extension provides preprocessing and external editing capabilities for Second Life's LSL (Linden Scripting Language) and SLua (Second Life Lua) scripts.
---

# Second Life External Scripting Extension - AI Assistant Guide

This VS Code extension provides preprocessing and external editing capabilities for Second Life's LSL (Linden Scripting Language) and SLua (Second Life Lua) scripts.

## Architecture Overview

### Core Service Pattern

The extension uses a **singleton service architecture** with three main services:

- `ConfigService` - Configuration management and settings
- `LanguageService` - Script preprocessing, mapping, and language features
- `SynchService` - WebSocket communication with Second Life viewer

All services follow the pattern: `Service.getInstance(context?)` and implement `vscode.Disposable`.

### Service Dependencies

```
SynchService → LanguageService → Preprocessor
               ↘
                ViewerEditWSClient
```

## Key Components

### Lexing-Based Preprocessor (2025-10 Addition)

**Location**: `src/shared/lexingpreprocessor.ts`

A new token-based preprocessor implementation is being developed alongside the existing regex-based preprocessor (`preprocessservice.ts`). This represents a modern, more accurate approach to preprocessing.

**Architecture**:
- **Lexer**: Tokenizes source code character-by-character with proper context awareness
- **Parser**: Processes token stream and handles directives
- **LexingPreprocessor**: High-level interface coordinating lexer and parser

**Key Design Principle - Language Agnostic**:

The lexer uses a **configuration-based approach** instead of hardcoded language checks:

```typescript
interface LanguageLexerConfig {
    lineCommentPrefix: string;        // e.g., "//" or "--" or "#"
    blockCommentStart: string | null; // e.g., "/*" or "--[[" or null
    blockCommentEnd: string | null;   // e.g., "*/" or "]]" or null
    directivePrefix: string;          // e.g., "#" or "--#" or "@"
    directiveKeywords: string[];      // e.g., ["include", "define", "ifdef"]
}
```

**Usage**:
```typescript
// Predefined language
const lexer = new Lexer(source, "lsl");  // or "luau"

// Custom language configuration
const customConfig: LanguageLexerConfig = {
    lineCommentPrefix: "#",
    blockCommentStart: null,
    blockCommentEnd: null,
    directivePrefix: "@",
    directiveKeywords: ["import", "define"],
};
const lexer = new Lexer(source, customConfig);
```

**Design Benefits**:
- ✅ No `if (language === "lsl")` conditionals in lexer code
- ✅ Easy to add support for new languages
- ✅ Users can define custom preprocessor syntax
- ✅ Cleaner, more testable code

**Documentation**:
- `doc/lexing-preprocessor.md` - Architecture overview
- `doc/lexing-preprocessor-language-config.md` - Configuration guide with examples
- `doc/lexing-preprocessor-quickstart.md` - Developer guide
- `doc/lexing-preprocessor-diagrams.md` - Visual architecture

**Current Status**: Tokenization complete with proper comment handling. Macro processor (`MacroProcessor`) and conditional processor (`ConditionalProcessor`) classes implemented.

**Testing**: `src/test/suite/lexingpreprocessor.test.ts` - 18 tests covering tokenization, comment handling, and custom configurations.

### Conditional Processor (2025-10 Addition)

**Location**: `src/shared/conditionalprocessor.ts`

A dedicated class for managing conditional compilation state in the lexing preprocessor. Replaces the simple array-based approach with a robust, stack-based processor.

**Architecture**:
- **ConditionalBlock**: State tracking for each `#if`/`#ifdef`/`#ifndef` block
- **Stack-based**: Maintains nested conditional blocks with proper parent/child relationships
- **Language-aware**: Constructed with `ScriptLanguage` parameter for future language-specific behavior

**Key Design Pattern - Encapsulation**:

Similar to `MacroProcessor`, the conditional processor encapsulates all state management:

```typescript
class ConditionalProcessor {
    // State queries
    isActive(): boolean              // Should code be included?
    getDepth(): number              // Current nesting depth
    hasUnclosedBlocks(): boolean    // Unmatched directives?

    // Directive processors
    processIfdef(macroName, macros, line): ConditionalResult
    processIfndef(macroName, macros, line): ConditionalResult
    processIf(tokens, macros, line): ConditionalResult
    processElif(tokens, macros, line): ConditionalResult
    processElse(line): ConditionalResult
    processEndif(line): ConditionalResult
}
```

**Conditional Logic**:
- Code is included only when **all** nested blocks are active
- In `#if`/`#elif`/`#else` chains, only the **first** satisfied branch is taken
- Parent block state determines whether children can be active
- Tracks whether any branch in a chain has been taken to prevent multiple activations

**Error Handling**:
All processors return `ConditionalResult` with:
- `success`: Whether directive was valid
- `shouldInclude`: Whether code should be included after this directive
- `message`: Error/warning text if applicable

**Integration**:
- `ParserState` uses `conditionals: ConditionalProcessor` instead of simple object
- Parser directive handlers delegate to processor methods
- Token emission checks `conditionals.isActive()` before output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secondlife/sl-vscode-plugin](https://github.com/secondlife/sl-vscode-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
