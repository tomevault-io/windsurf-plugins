---
trigger: always_on
description: TITLE: Implement Transport Error Handling in TypeScript and Python
---

TITLE: Implement Transport Error Handling in TypeScript and Python
DESCRIPTION: This code snippet illustrates how to incorporate comprehensive error handling within transport implementations. It demonstrates catching exceptions during connection establishment and message transmission, logging errors, and ensuring proper resource cleanup using `try-catch` blocks and context managers.
SOURCE: https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs/concepts/transports.mdx#_snippet_4

LANGUAGE: TypeScript
CODE:
```
class ExampleTransport implements Transport {
  async start() {
    try {
      // Connection logic
    } catch (error) {
      this.onerror?.(new Error(`Failed to connect: ${error}`));
      throw error;
    }
  }

  async send(message: JSONRPCMessage) {
    try {
      // Sending logic
    } catch (error) {
      this.onerror?.(new Error(`Failed to send message: ${error}`));
      throw error;
    }
  }
}
```

LANGUAGE: Python
CODE:
```
@contextmanager
async def example_transport(scope: Scope, receive: Receive, send: Send):
    try:
        # Create streams for bidirectional communication
        read_stream_writer, read_stream = anyio.create_memory_object_stream(0)
        write_stream, write_stream_reader = anyio.create_memory_object_stream(0)

        async def message_handler():
            try:
                async with read_stream_writer:
                    # Message handling logic
                    pass
            except Exception as exc:
                logger.error(f"Failed to handle message: {exc}")
                raise exc

        async with anyio.create_task_group() as tg:
            tg.start_soon(message_handler)
            try:
                # Yield streams for communication
                yield read_stream, write_stream
            except Exception as exc:
                logger.error(f"Transport error: {exc}")
                raise exc
            finally:
                tg.cancel_scope.cancel()
                await write_stream.aclose()
                await read_stream.aclose()
    except Exception as exc:
        logger.error(f"Failed to initialize transport: {exc}")
        raise exc
```

----------------------------------------

TITLE: Implement Tool Execution Handler with Weather API Tools (Kotlin)
DESCRIPTION: This snippet demonstrates how to set up an HTTP client using Ktor for making requests to the weather.gov API and how to register two tools (`get_alerts` and `get_forecast`) with an MCP server. The `get_alerts` tool fetches weather alerts by state, validating the 'state' parameter. The `get_forecast` tool retrieves weather forecasts by latitude and longitude, validating both parameters. Both tools handle input validation and return `CallToolResult` with `TextContent`.
SOURCE: https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/quickstart/server.mdx#_snippet_33

LANGUAGE: kotlin
CODE:
```
// Create an HTTP client with a default request configuration and JSON content negotiation
val httpClient = HttpClient {
    defaultRequest {
        url("https://api.weather.gov")
        headers {
            append("Accept", "application/geo+json")
            append("User-Agent", "WeatherApiClient/1.0")
        }
        contentType(ContentType.Application.Json)
    }
    // Install content negotiation plugin for JSON serialization/deserialization
    install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
}

// Register a tool to fetch weather alerts by state
server.addTool(
    name = "get_alerts",
    description = """
        Get weather alerts for a US state. Input is Two-letter US state code (e.g. CA, NY)
    """.trimIndent(),
    inputSchema = Tool.Input(
        properties = buildJsonObject {
            putJsonObject("state") {
                put("type", "string")
                put("description", "Two-letter US state code (e.g. CA, NY)")
            }
        },
        required = listOf("state")
    )
) { request ->
    val state = request.arguments["state"]?.jsonPrimitive?.content
    if (state == null) {
        return@addTool CallToolResult(
            content = listOf(TextContent("The 'state' parameter is required."))
        )
    }

    val alerts = httpClient.getAlerts(state)

    CallToolResult(content = alerts.map { TextContent(it) })
}

// Register a tool to fetch weather forecast by latitude and longitude
server.addTool(
    name = "get_forecast",
    description = """
        Get weather forecast for a specific latitude/longitude
    """.trimIndent(),
    inputSchema = Tool.Input(
        properties = buildJsonObject {
            putJsonObject("latitude") { put("type", "number") }
            putJsonObject("longitude") { put("type", "number") }
        },
        required = listOf("latitude", "longitude")
    )
) { request ->
    val latitude = request.arguments["latitude"]?.jsonPrimitive?.doubleOrNull
    val longitude = request.arguments["longitude"]?.jsonPrimitive?.doubleOrNull
    if (latitude == null || longitude == null) {
        return@addTool CallToolResult(
            content = listOf(TextContent("The 'latitude' and 'longitude' parameters are required."))
        )
    }

    val forecast = httpClient.getForecast(latitude, longitude)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kerlos/elysia-mcp](https://github.com/kerlos/elysia-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
