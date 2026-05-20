---
trigger: always_on
description: fast-mcp-scala is a high-level Scala 3 library for building Model Context Protocol (MCP) servers. It provides two registration paths:
---

# CLAUDE.md - fast-mcp-scala Development Guide

## Project Overview

fast-mcp-scala is a high-level Scala 3 library for building Model Context Protocol (MCP) servers. It provides two registration paths:

1. **Annotation-driven** (`@Tool`, `@Resource`, `@Prompt` + `scanAnnotations`) — zero-boilerplate on JVM and Scala.js/Bun
2. **Typed contracts** (`McpTool`, `McpPrompt`, `McpStaticResource`, `McpTemplateResource`) — explicit, cross-platform (JVM + Scala.js)

Both paths converge on the same `McpServer` trait and support `@Param` metadata on parameters/fields.

## Build System

**Build tool**: Mill 1.1.5 (configured in `.mill-version`)
**Scala**: 3.8.3
**Plugins**: mill-bun-plugin 0.2.0 (Scala.js + Bun integration)

### Common Commands

```bash
# Aggregates (run across JVM + Scala.js)
./mill fast-mcp-scala.compile                       # Compile all platforms
./mill fast-mcp-scala.test                          # All tests (JVM + Bun conformance)
./mill fast-mcp-scala.reformat                      # Auto-format every Scala source
./mill fast-mcp-scala.checkFormat                   # Scalafmt check (CI uses this)

# Single-platform
./mill fast-mcp-scala.jvm.test                      # JVM tests only
./mill fast-mcp-scala.js.test.bunTest               # Scala.js conformance tests only
./mill fast-mcp-scala.jvm.test com.tjclp.fastmcp.macros.ToolProcessorTest

# Publish
./mill fast-mcp-scala.jvm.publishLocal              # Publish JVM artifact to ~/.ivy2/local
./mill fast-mcp-scala.js.publishLocal               # Publish Scala.js artifact to ~/.ivy2/local
./mill -i __.publishLocal                           # Publish both artifacts
```

## Project Structure

```
fast-mcp-scala/
├── build.mill                 # Mill build definition
├── .mill-version              # Mill version (1.1.5)
├── fast-mcp-scala/
│   ├── shared/src/            # Platform-independent code (JVM + JS)
│   │   └── com/tjclp/fastmcp/
│   │       ├── core/
│   │       │   ├── Annotations.scala    # @Tool, @Param, @Resource, @Prompt
│   │       │   ├── Types.scala          # ToolDefinition, Content, ToolInputSchema, etc.
│   │       │   └── Contracts.scala      # McpTool, McpPrompt, McpDecoder, McpEncoder
│   │       ├── runtime/                 # RefResolver
│   │       └── server/
│   │           ├── McpServerCore.scala  # McpServerCore trait (abstract API)
│   │           ├── McpContext.scala     # Platform-independent context base
│   │           ├── McpServerSettings.scala
│   │           └── manager/            # ToolManager, PromptManager, ResourceManager
│   ├── jvm/
│   │   ├── src/               # JVM-specific code
│   │   │   └── com/tjclp/fastmcp/
│   │   │       ├── core/Types.scala         # TypeConversions (toJava extensions, private[fastmcp])
│   │   │       ├── macros/                  # JVM-side macro/runtime support
│   │   │       │   ├── ToolProcessor.scala
│   │   │       │   ├── ResourceProcessor.scala
│   │   │       │   ├── PromptProcessor.scala
│   │   │       │   ├── RegistrationMacro.scala  # scanAnnotations entry point
│   │   │       │   ├── JsonSchemaMacro.scala
│   │   │       │   ├── JacksonConverter.scala   # extends McpDecoder (bridges to shared)
│   │   │       │   └── JacksonConversionContext.scala  # extends McpDecodeContext
│   │   │       ├── server/
│   │   │       │   ├── FastMcpServer.scala      # JVM implementation (extends McpServerCore)
│   │   │       │   ├── McpContext.scala         # JvmMcpContext (private[fastmcp])
│   │   │       │   ├── McpServerBuilders.scala  # McpServer companion (factory methods)
│   │   │       │   └── transport/
│   │   │       └── examples/
│   │   └── test/src/          # JVM test sources
│   └── js/                    # Scala.js code (Bun-first runtime)
│       ├── src/               # JsMcpServer, TS SDK facades, Bun runtime, examples
│       └── test/src/          # Conformance, HTTP, codec, contract surface tests
```

## Key Concepts

### Annotation Path (JVM + Scala.js/Bun)

```scala
object MyServer extends ZIOAppDefault:
  @Tool(name = Some("add"), description = Some("Add two numbers"))
  def add(@Param("First number") a: Int, @Param("Second number") b: Int): Int = a + b

  override def run =
    for
      server <- ZIO.succeed(McpServer("MyServer"))
      _ <- ZIO.attempt(server.scanAnnotations[MyServer.type])
      _ <- server.runStdio()
    yield ()
```

### Typed Contract Path (cross-platform)

```scala
case class AddArgs(@Param("First number") a: Int, @Param("Second number") b: Int)

val addTool = McpTool.derived[AddArgs, Int](
  name = "add",
  description = Some("Add two numbers")
) { args => ZIO.succeed(args.a + args.b) }

// Mount:
server.tool(addTool)
```

### When to Use Which

| | Annotations | Typed Contracts |
|---|---|---|
| Platform | JVM only | JVM + Scala.js |
| Boilerplate | Zero (macro-driven) | Minimal (case class + builder) |
| Schema | Auto from method signature | Auto from case class via `ToolSchemaProvider` on JVM and JS |
| `@Param` | On method parameters | On case class fields |
| Composability | Methods on an object | First-class values |
| Best for | Quick servers, prototyping | Libraries, cross-platform, production |

### Annotations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TJC-LP/fast-mcp-scala](https://github.com/TJC-LP/fast-mcp-scala) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
