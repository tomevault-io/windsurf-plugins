---
trigger: always_on
description: **If you don't know something, DO NOT make it up. Either:**
---

# BoxLang AI Module - AI Agent Instructions

## ⚠️ CRITICAL: Do Not Hallucinate

**If you don't know something, DO NOT make it up. Either:**

1. **Ask the user** for clarification
2. **Search the codebase** using available tools (grep_search, semantic_search, read_file)
3. **Check the actual BIF files** in `src/main/bx/bifs/` to verify function names and signatures
4. **Acknowledge uncertainty** - It's better to say "I'm not sure, let me check" than to provide incorrect information

**Never assume:**

- Function names or BIF names that you haven't verified
- API signatures or parameter names
- Class names or file locations
- Feature availability

**Always verify before suggesting code that uses:**

- BIFs (Built-in Functions) - check `src/main/bx/bifs/*.bx`
- Classes - check `src/main/bx/models/`
- Configuration options - check actual source files

**Prefer simplicity/pragmatism over complexity**

## Project Overview

This is a **BoxLang module** providing unified AI provider integration. BoxLang is a modern dynamic JVM language (CFML-like syntax) with Java interop. The module exposes **Built-in Functions (BIFs)** written in BoxLang that interface with multiple AI providers (OpenAI, Claude, Gemini, Ollama, etc.) through a consistent API.

**Key Architecture:**

- **Hybrid codebase**: BoxLang (`.bx` files) for business logic + Java for runtime integration
- **Module structure**: `src/main/bx/` contains BoxLang source, compiled into `build/module/` for distribution
- **Provider pattern**: All AI services extend `BaseService` (OpenAI-compatible) implementing `IAiService` interface
- **Runnable pipelines**: Composable AI operations via `IAiRunnable` interface (models, messages, transformers)

## BoxLang Language Conventions

### Syntax Essentials

```java
// BoxLang looks like Java/CFML hybrid
class extends="BaseClass" implements="IInterface" {
    property name="field" type="string" default="";

    function methodName( required arg, optional param = "default" ) {
        return this;  // Fluent APIs are common
    }
}

// Imports - CRITICAL: Classes ALWAYS require imports defined at the top of the class definition
// Do NOT use inline imports inside methods/functions, that can only be used in scripts (bxs) or (bxm)
import bxModules.bxai.models.util.TextChunker;

// Call static methods using :: operator
result = TextChunker::chunk( text, options )

// Static variables must be referenced via static scope
static {
    DEFAULT_OPTIONS = { key: "value" };
}

function someMethod() {
    var config = static.DEFAULT_OPTIONS; // Must use static. prefix
}

// Struct append() without duplicate
var merged = sourceStruct.append( defaultStruct, false ); // false = no override

// Null-safe navigation and Elvis operator
result = service?.invoke( request ) ?: "default"

// Array/struct operations (dynamic and functional)
messages.map( m => m.content ).filter( c => !isNull(c) )
```

### Key Differences from Java

- **No semicolons required** (but allowed)
- **Duck typing**: `any` type allows dynamic dispatch
- **Built-in serialization**: `jsonSerialize()`, `jsonDeserialize()` (NOT serializeJSON/deserializeJSON)
- **Implicit returns**: Last expression in function is returned
- **String interpolation**: `"Hello, ${name}!"` or `"#name#"`
- **OnMissingMethod**: Dynamic method handling (see `AiMessage` for roled messages)
- **Implicit getters/setters**: Properties declared with `property` automatically get getter/setter methods
  - `property name="serverName"` → `getServerName()` and `setServerName(value)` are auto-generated
  - Do NOT manually create getter/setter methods for properties
  - Access via `obj.getPropertyName()` or `obj.setPropertyName(value)`
- **Rich string functions**: Comprehensive string manipulation BIFs + full Java String API access
  - Reference: https://boxlang.ortusbooks.com/boxlang-language/reference/built-in-functions/string
  - Examples: `char(10)` (newline), `left()`, `right()`, `reReplace()`, `trim()`, etc.

### Code Quality Standards

- **No cryptic variable names**: Use descriptive, self-documenting names (e.g., `maxConnections` not `M`)
- **Avoid acronyms**: Only use acronyms that are universally known (HTTP, URL, API). Prefer full words.
- **Avoid reserved scope names**: BoxLang has built-in scopes that cannot be used as variable names:
  - `server`, `request`, `session`, `application`, `cgi`, `url`, `form`, `cookie`, `variables`
  - Use alternative names like `mcpSrv`, `rpcRequest`, `httpReq`, etc.
- **Type casting**: Use `castAs` operator instead of `javaCast()` function

  ```java
  // Good
  arguments.config.diversityFactor castAs "float"
  arguments.config.diversityFactor castAs float

  // Bad
  javaCast( "float", arguments.config.diversityFactor )
  ```

## Development Workflows

### Build & Test

```bash
# Full build (downloads BoxLang runtime, compiles module, runs tests)
./gradlew build

# Skip tests during development
./gradlew shadowJar -x test

# Run specific test class
./gradlew test --tests "ortus.boxlang.ai.bifs.aiMessageTest"

# Start Ollama for local testing
docker compose up -d ollama
curl http://localhost:11434/api/tags  # Verify model availability
```

### Module Development Cycle

1. Edit BoxLang source in `src/main/bx/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ortus-boxlang/bx-ai](https://github.com/ortus-boxlang/bx-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
