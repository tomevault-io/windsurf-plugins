---
trigger: always_on
description: This guide covers how to create custom WebMCP tools that run in the browser context and are available to the LLM for execution.
---

# WebMCP Tool Development Guide

This guide covers how to create custom WebMCP tools that run in the browser context and are available to the LLM for execution.

> **Spec Alignment**: This implementation aligns with the [WebMCP proposed spec](https://github.com/webmachinelearning/webmcp/blob/main/docs/proposal.md).

## Basic Structure

Every WebMCP tool is a JavaScript file with two required exports:

```javascript
'use webmcp-tool v1';

export const metadata = {
  name: 'tool_name', // Snake_case, unique within namespace
  namespace: 'my_namespace', // Groups related tools, appears in tool ID
  version: '1.0.0', // Semver
  description: 'What this tool does. Be specific - the LLM reads this.',
  match: 'https://example.com/*', // URL pattern(s) where tool is available
  inputSchema: {
    type: 'object',
    properties: {
      // JSON Schema for tool arguments
    },
    additionalProperties: false,
  },
};

// Per WebMCP spec: execute receives (args, agent) where agent provides requestUserInteraction()
export async function execute(args = {}, agent) {
  // Tool implementation - has full DOM/window access
  // Use agent.requestUserInteraction() for user confirmation flows
  // Return result object
}
```

The tool ID becomes `{namespace}_{name}` (e.g., `myapp_get_context`).

## Metadata Fields

### `name` (required)

- Use `snake_case`
- Should describe the tool's function: `document_context`, `get_messages`, `extract_data`

### `namespace` (required)

- Groups related tools: `myapp`, `github`, `jira`, `notion`
- Helps LLM understand tool's domain

### `description` (required)

The LLM reads this to decide when to use your tool. A good description answers three questions:

1. **What does it do?** Be specific about the action and target
2. **What context does it provide?** Mention key capabilities (e.g., "with timestamps", "with author names")
3. **Any constraints?** Note requirements that affect success (e.g., "Requires captions")

**Best practices:**

- Describe capabilities, not routing. Tool selection priority is handled by the system prompt via `<site_tools>`.
- Keep under 200 characters. The description appears inline in `<site_tools>` hints.
- No need to list response fields. The model sees the output.

**Good examples:**

```
"Fetch Slack conversation from the current channel, DM, or thread with author names, timestamps, reactions, and thread replies."

"Extract content, selection, comments, and structure from the current Google Doc."
```

**Bad:** "Gets app data" (vague, no capability info)

### `match` (required)

URL patterns where the tool should be available:

```javascript
// Single pattern (string)
match: 'https://app.example.com/*';

// Multiple patterns (array)
match: ['*://www.example.com/view*', '*://example.com/view*'];

// All URLs (use sparingly)
match: ['<all_urls>'];
```

Pattern syntax:

- `*` matches any characters
- `://` separates scheme from host
- Use specific patterns to avoid injecting tools where they don't apply

### `inputSchema` (required)

JSON Schema defining tool arguments:

```javascript
inputSchema: {
  type: "object",
  properties: {
    limit: {
      type: "number",
      description: "Maximum items to fetch.",
      default: 100
    },
    format: {
      type: "string",
      enum: ["full", "summary"],
      description: "Output format. Default: full"
    },
    includeMetadata: {
      type: "boolean",
      description: "Include extra metadata in response"
    }
  },
  required: ["selector"],  // List required fields
  additionalProperties: false
}
```

If no arguments needed:

```javascript
inputSchema: {
  type: "object",
  properties: {},
  additionalProperties: false
}
```

## The `execute` Function

Per the WebMCP spec, execute receives two arguments:

- `args` - The tool arguments from the LLM
- `agent` - An agent context object with `requestUserInteraction()` for user confirmation flows

```javascript
export async function execute(args = {}, agent) {
  // Destructure with defaults
  const { limit = 100, format = 'full' } = args;

  // Tool logic here...

  // Return result
  return { ... };
}
```

### Requesting User Interaction

For tools that perform sensitive actions (purchases, deletions, etc.), use `agent.requestUserInteraction()`:

```javascript
export async function execute(args = {}, agent) {
  const { productId } = args;

  // Request user confirmation before sensitive action
  const confirmed = await agent.requestUserInteraction(async () => {
    return confirm(`Purchase product ${productId}?\nClick OK to confirm.`);
  });

  if (!confirmed) {
    throw new Error('Purchase cancelled by user.');
  }

  // Proceed with action...
  await executePurchase(productId);
  return { success: true, productId };
}
```

The `requestUserInteraction` API:

- Takes an async function that performs the UI interaction
- Returns the result of that function
- Allows tools to prompt for confirmation, input, or any other user interaction
- The agent (browser) handles pausing execution while waiting for user input

### Available in `execute`:

- Full DOM access: `document`, `window`
- Current URL: `window.location`
- Fetch API: `fetch()` (with page's cookies/auth)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igrigorik/AgentBoard](https://github.com/igrigorik/AgentBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
