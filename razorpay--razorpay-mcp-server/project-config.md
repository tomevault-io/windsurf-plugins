---
trigger: always_on
description: This file provides instructions for AI coding agents working on this repository.
---

# Agent Instructions

This file provides instructions for AI coding agents working on this repository.

## Project Overview

This is a Go MCP (Model Context Protocol) server that wraps Razorpay APIs. Tools live in `pkg/razorpay/`, are registered in `pkg/razorpay/tools.go`, and tested alongside in `*_test.go` files.

## Key Commands

```bash
make test    # Run all tests
make fmt     # Format code
make lint    # Run golangci-lint
make build   # Build the binary
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for branch naming (`username/feature`), commit format (`[type]: description`), and PR process.

## Code Style

- 80-char line length max (lll linter)
- `goimports` with local prefix `github.com/razorpay/razorpay-mcp-server`
- Full linter config in `.golangci.yaml`

---

# Razorpay MCP Tool Generator

Generate complete MCP tool implementations from Razorpay API docs, curl commands, or request/response examples.

## Input Formats

The user may provide any of:

1. **Razorpay docs URL** — `https://razorpay.com/docs/api/...`
2. **curl command** — extract method, path, headers, body, and response
3. **Request/response JSON** — raw payload and expected response
4. **SDK function signature** — Go SDK function to call

If the **tool name** is not provided or cannot be inferred, **ask the user** before proceeding.

## Workflow

```
Task Progress:
- [ ] Step 1: Parse input and extract API contract
- [ ] Step 2: Determine tool name and resource file
- [ ] Step 3: Implement tool function
- [ ] Step 4: Register tool in tools.go
- [ ] Step 5: Write unit tests
- [ ] Step 6: Update README.md Available Tools table
- [ ] Step 7: Run linter — fix errors
- [ ] Step 8: Run tests — fix errors
- [ ] Step 9: Create branch, commit, and open PR (if gh available)
```

### Step 1: Parse Input

**From docs URL**: Fetch the page and extract endpoint path, HTTP method, required/optional params with types and descriptions, and example response.

**From curl**: Parse the method (`-X`), URL path, headers (`-H`), request body (`-d`), and note the response if provided.

**From request/response JSON**: Infer required vs optional fields, types, and construct parameter definitions.

Map each parameter to a validator type:

| JSON type | Parameter helper | Validator method |
|-----------|-----------------|------------------|
| string | `mcpgo.WithString` | `ValidateAndAddRequiredString` / `ValidateAndAddOptionalString` |
| number (int) | `mcpgo.WithNumber` | `ValidateAndAddRequiredInt` / `ValidateAndAddOptionalInt` |
| number (float) | `mcpgo.WithNumber` | `ValidateAndAddRequiredFloat` / `ValidateAndAddOptionalFloat` |
| boolean | `mcpgo.WithBoolean` | `ValidateAndAddRequiredBool` / `ValidateAndAddOptionalBool` |
| object | `mcpgo.WithObject` | `ValidateAndAddRequiredMap` / `ValidateAndAddOptionalMap` |
| array | `mcpgo.WithArray` | `ValidateAndAddRequiredArray` / `ValidateAndAddOptionalArray` |

For nested objects (e.g., `customer.name` flattened to `customer_name`), use `ValidateAndAddOptionalStringToPath`.

### Step 2: Determine Tool Name and File

Naming conventions:
- Fetch single: `fetch_{resource}` → `Fetch{Resource}`
- Fetch list: `fetch_all_{resources}` → `FetchAll{Resources}`
- Create: `create_{resource}` → `Create{Resource}`
- Update: `update_{resource}` → `Update{Resource}`

Place the tool in `pkg/razorpay/{resource_type}.go`. Create a new file only if the resource type doesn't already exist.

### Step 3: Implement Tool

Follow this exact structure:

```go
func ToolName(
	obs *observability.Observability,
	client *rzpsdk.Client,
) mcpgo.Tool {
	parameters := []mcpgo.ToolParameter{
		// Required params first, then optional
	}

	handler := func(
		ctx context.Context,
		r mcpgo.CallToolRequest,
	) (*mcpgo.ToolResult, error) {
		client, err := getClientFromContextOrDefault(ctx, client)
		if err != nil {
			return mcpgo.NewToolResultError(err.Error()), nil
		}

		payload := make(map[string]interface{})
		validator := NewValidator(&r).
			ValidateAndAddRequiredString(payload, "id")
			// chain more validators...

		if result, err := validator.HandleErrorsIfAny(); result != nil {
			return result, err
		}

		response, err := client.Resource.Method(/* args */)
		if err != nil {
			return mcpgo.NewToolResultError(
				fmt.Sprintf("operation failed: %s", err.Error())), nil
		}

		return mcpgo.NewToolResultJSON(response)
	}

	return mcpgo.NewTool("tool_name", "description", parameters, handler)
}
```

#### Writing LLM-Friendly Tool Descriptions

The description string in `mcpgo.NewTool` is what LLMs read to decide which tool to call. A bad description means the tool gets ignored or misused. Every description **must** answer three questions:

1. **What** does this tool do?
2. **When** should an LLM pick this tool? (trigger conditions, prerequisites)
3. **What** constraints or gotchas should the LLM know? (units, required states, return format)

**Structure** (2-4 sentences):

```
[Action verb] + [what it does] + [key context].
[When to use / prerequisites]. [Constraints, units, or return format].
```

**Bad examples** (too vague, no context):

```go
// Vague — LLM doesn't know when to pick this
"Fetch an order's details using its ID"

// No constraints — LLM won't know about paise

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [razorpay/razorpay-mcp-server](https://github.com/razorpay/razorpay-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
