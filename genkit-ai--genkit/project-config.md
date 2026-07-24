---
trigger: always_on
description: As an AI developer working on this codebase, you MUST adhere to these operational protocols:
---

# Go Development Guidelines

## Agent Instructions

As an AI developer working on this codebase, you MUST adhere to these operational protocols:

* **Registry-First Thinking**: Genkit is built around a central registry. Always use `genkit.DefineX` (e.g., `DefineFlow`, `DefineTool`) for any component that should be discoverable by tools or the Dev UI.
* **Context is Mandatory**: Every Genkit function and tool execution requires a `context.Context`. Pass it through faithfully; never use `context.Background()` inside an implementation unless it is the entry point.
* **Schema Strictness**: When defining input/output types for Flows or Tools, use clear, JSON-tagged Go structs. The model uses these tags and types to understand the interface.
* **Search Before Implementing**: Many common utilities exist in `internal/base` or `ai/`. Check there before implementing new JSON parsing or string manipulation logic.
* **Idiomatic Concurrency**: Prefer Go's native concurrency (goroutines/channels) but be mindful of the `context` lifecycle within long-running background tasks.

## Architecture & Patterns

* **Package Structure**:
  * `genkit`: The main entry point for application developers. Contains high-level functions for defining flows, prompts, and tools.
  * `ai`: Contains the core AI types and interfaces (Model, Prompt, Tool, Embedder, Retriever, etc.).
  * `core`: Contains the underlying framework primitives (Actions, Flows, Tracing, Registry). Mostly for internal use or plugin development.
  * `plugins`: Contains implementations for specific providers (Google AI, Vertex AI, Ollama, etc.).
  * `internal`: Contains private implementations, shared utilities, and development tools.
    * `base`: Low-level utilities for JSON (extraction, normalization, validation) and context handling.
    * `cmd`: Internal command-line tools for code generation (`jsonschemagen`), file synchronization (`copy`), and documentation (`weave`).
    * `registry`: Implementation of the Genkit action and schema registry.
    * `metrics`: OpenTelemetry instrumentation for framework metrics.
    * `fakeembedder`: Testing utilities for simulating embedding providers.
* **Constructors**:
  * **`DefineX`** (e.g., `DefineFlow`, `DefineTool`): Creates **and registers** a component with the registry. Use this for components that should be discoverable by the reflection API (and thus the Dev UI).
  * **`NewX`** (e.g., `NewTool`): Creates a component **without registering it**. Use this for internal components, dynamic creation, or testing.
* **Options Pattern**:
  * Use functional options (e.g., `ai.WithModel`, `genkit.WithPlugins`) for optional configuration parameters in constructors and method calls. This maintains API stability while allowing for expansion.

## Canonical Coding Patterns

### Defining a Flow
Use `genkit.DefineFlow` for orchestration tasks. Use typed structs for input and output to ensure schema generation.

```go
type OrderInput struct {
	ID int `json:"id"`
}

type OrderOutput struct {
	Status string `json:"status"`
}

// Define the flow
var GetOrderStatusFlow = genkit.DefineFlow(g, "getOrderStatus",
	func(ctx context.Context, input *OrderInput) (*OrderOutput, error) {
		// Use genkit.Run to create a trace span for a specific step
		status, err := genkit.Run(ctx, "lookup-db", func() (string, error) {
			return "Shipped", nil 
		})
		if err != nil {
			return nil, err
		}
		return &OrderOutput{Status: status}, nil
	},
)
```

### Defining a Tool
Tools are used by models. The description is crucial as the model uses it to decide when to call the tool.

```go
type WeatherInput struct {
	Location string `json:"location"`
}

var WeatherTool = genkit.DefineTool(g, "getWeather", "fetches current weather for a location",
	func(ctx *ai.ToolContext, input *WeatherInput) (string, error) {
		// Implementation logic
		return "Sunny", nil
	},
)
```

### Generation with Typed Data
Use `genkit.GenerateData` to get structured output from a model directly into a Go struct.

```go
type Recipe struct {
	Name        string   `json:"name"`
	Ingredients []string `json:"ingredients"`
}

recipe, _, err := genkit.GenerateData[Recipe](ctx, g,
	ai.WithModelName("googleai/gemini-2.0-flash"),
	ai.WithPrompt("Suggest a pancake recipe"),
)
```

## Code Quality & Linting

* **Run Linting**: Always run `go vet ./...` from the `go/` directory for all Go code changes.
* **Format Code**: Run `bin/fmt` (which runs `go fmt`) to ensure code is formatted correctly.
* **Pass All Tests**: Ensure all unit tests pass (`go test ./...`).
* **Production Ready**: The objective is to produce production-grade code.
* **Shift Left**: Employ a "shift left" strategy—catch errors early.
* **Strict Typing**: Go is statically typed. Do not use `interface{}` (or `any`) unless absolutely necessary and documented.
* **No Warning Suppression**: Avoid ignoring linter warnings unless there is a compelling, documented reason.
* Group imports: standard library first, then third-party, then internal. `goimports` handles this automatically.

## Generated Files & Data Model

* **Do Not Edit Generated Files**: Files generated by tools (like protobufs or strict-type generators) should not be modified directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genkit-ai/genkit](https://github.com/genkit-ai/genkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
