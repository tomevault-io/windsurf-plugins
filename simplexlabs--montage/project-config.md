---
trigger: always_on
description: Reference for the Simplex browser automation CLI. Use when the user mentions Simplex, simplex CLI, simplex editor, browser automation workflows, or wants to create/run/stream browser sessions. Also use when you see simplex commands or SIMPLEX_API_KEY.
---


# Simplex CLI Reference

Simplex is a browser automation platform. Users define workflows (name + URL + optional variables), then run them as browser sessions controlled by an AI agent. The CLI lets you create workflows, start sessions, poll for events, and send messages to the agent.

## Installation

```bash
pip install simplex
```

This installs the `simplex` CLI command.

## Authentication

```bash
# Option 1: Environment variable
export SIMPLEX_API_KEY="your-api-key"

# Option 2: Login command — opens browser (saves to ~/.simplex/credentials)
simplex login

# Option 3: Direct API key (for CI/headless)
simplex login --api-key sk-...
```

To point at a different environment:
```bash
export SIMPLEX_BASE_URL="https://your-custom-api-url.example.com"
```

## CLI Commands

### `simplex editor` — Create a workflow and start an interactive editor session

```bash
simplex editor --name "My Workflow" --url "https://example.com"
simplex editor -n "Test" -u "https://example.com" --vars '{"email":"a@b.com","name":"Test"}'
simplex editor -n "Test" -u "https://example.com" --vars data.json
simplex editor --name "Test" --url "https://example.com" --json
```

| Flag | Short | Required | Description |
|------|-------|----------|-------------|
| `--name` | `-n` | Yes | Workflow name |
| `--url` | `-u` | Yes | Starting URL |
| `--vars` | | No | Variables as inline JSON string or path to a .json file |
| `--json` | | No | Output session info as JSON (for programmatic use) |

Creates a workflow and starts a browser session. Returns immediately with workflow ID, session ID, VNC URL, and logs URL.

### `simplex send` — Send a message to a running session

```bash
simplex send <workflow_id> "Click the login button"
simplex send <workflow_id> "Fill in the email"
```

| Argument | Description |
|----------|-------------|
| `workflow_id` | **Required.** Workflow ID |
| `message` | **Required.** Message to send to the browser agent |

Looks up the active session for the workflow and sends the message to the browser agent.

### `simplex interrupt` — Interrupt a running editor session

```bash
simplex interrupt <workflow_id>
```

Takes a workflow ID, looks up the active session, and sends an interrupt signal to the agent.

### `simplex run` — Run an existing workflow

```bash
simplex run <workflow_id>
simplex run <workflow_id> --vars '{"email":"test@test.com","zip":"91711"}' --watch
simplex run <workflow_id> --vars variables.json --watch
```

| Flag | Short | Description |
|------|-------|-------------|
| `--vars` | | Variables as inline JSON string or path to a .json file |
| `--metadata` | `-m` | Metadata string |
| `--webhook-url` | | Webhook URL for status updates |
| `--watch` | `-w` | Poll until completion |

### `simplex pause` / `simplex resume`

```bash
simplex pause <workflow_id>
simplex resume <workflow_id>
```

### `simplex workflows list`

```bash
simplex workflows list                       # List all workflows
simplex workflows list --name "search term"  # Filter by name
simplex workflows list --metadata "filter"   # Filter by metadata
```

### `simplex workflows vars` — Show variable schema for a workflow

```bash
simplex workflows vars <workflow_id>
simplex workflows vars <workflow_id> --json
```

Displays a table of the workflow's variable definitions including name, type, whether it's required, default value, and allowed enum values.

### `simplex workflows outputs` — View structured output schema

```bash
simplex workflows outputs <workflow_id>
simplex workflows outputs <workflow_id> --json
```

Displays the structured output fields defined for a workflow — name, type, and description.

### `simplex workflows set-outputs` — Set structured output schema

```bash
# Add fields inline
simplex workflows set-outputs <workflow_id> --field title:string --field price:number

# With descriptions (name:type:description)
simplex workflows set-outputs <workflow_id> \
  --field "company_name:string:Name of the company" \
  --field "revenue:number:Annual revenue in USD" \
  --field "is_public:boolean:Whether publicly traded"

# Enum type (name:enum:value1,value2,value3)
simplex workflows set-outputs <workflow_id> \
  --field "status:enum:pending,active,closed" \
  --field "category:enum:tech,healthcare,finance"

# From a JSON file
simplex workflows set-outputs <workflow_id> --file schema.json

# Clear all outputs
simplex workflows set-outputs <workflow_id> --clear
```

| Flag | Short | Description |
|------|-------|-------------|
| `--field` | `-f` | Field as `name:type` or `name:type:description`. Repeatable. For enum: `name:enum:val1,val2` |
| `--file` | | Path to a JSON file containing the schema array |
| `--clear` | | Remove all structured outputs |
| `--json` | | Output raw JSON response |

**Supported types:** `string`, `number`, `boolean`, `array`, `object`, `enum`

**JSON file format** (same schema as the API):
```json
[
  {"name": "company_name", "type": "string", "description": "Name of the company"},
  {"name": "revenue", "type": "number", "description": "Annual revenue in USD"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simplexlabs/montage](https://github.com/simplexlabs/montage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
