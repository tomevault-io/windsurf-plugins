---
trigger: always_on
description: Declarative "commerce as code" CLI for Saleor. Define store configuration in YAML, sync with your Saleor instance.
---

# Saleor Configurator

Declarative "commerce as code" CLI for Saleor. Define store configuration in YAML, sync with your Saleor instance.

## Build, Test, and Development

```bash
pnpm install          # Install dependencies
pnpm build            # Compile TypeScript
pnpm test             # Run test suite
pnpm dev <command>    # Run CLI in development mode
pnpm check:fix        # Auto-fix lint and formatting
pnpm check:ci         # CI validation mode
```

## Project Structure

Primary source in `src/`. CLI surface (`src/cli`) wires Commander commands from `src/commands`. Orchestration in `src/core`. Entity logic in `src/modules/<entity>`. Shared tooling (GraphQL, logging, schemas) in `src/lib`. Tests mirror the structure. Documentation in `docs/`, YAML samples in `recipes/`.

## Commands

| Command | Purpose | Requires Network |
|---------|---------|-----------------|
| `validate` | Check config.yml against schema | No |
| `diff` | Compare local config with remote | Yes |
| `deploy --plan` | Preview deployment changes | Yes |
| `deploy` | Apply configuration to Saleor | Yes |
| `introspect` | Pull remote config to local file | Yes |

## Exit Codes

| Code | Name | Meaning |
|------|------|---------|
| 0 | SUCCESS | Operation completed |
| 1 | UNEXPECTED | Unknown error |
| 2 | AUTHENTICATION | Invalid or expired credentials |
| 3 | NETWORK | Connection failure |
| 4 | VALIDATION | Config schema errors |
| 5 | PARTIAL_FAILURE | Some operations failed |
| 6 | DELETION_BLOCKED | `--fail-on-delete` triggered |
| 7 | BREAKING_BLOCKED | `--fail-on-breaking` triggered |

## Safety Rules

- Always run `validate` before `diff` or `deploy`
- Always run `diff` or `deploy --plan` before `deploy`
- Use `--fail-on-delete` in CI/CD to prevent accidental deletions
- Interactive prompts are auto-skipped in non-TTY environments (no flag needed)

## Credentials Setup

Create `.env.local` in the project root (auto-loaded by the CLI):

```env
SALEOR_URL=https://your-store.saleor.cloud/graphql/
SALEOR_TOKEN=your-api-token
```

Or pass via flags:

```bash
pnpm dlx @saleor/configurator deploy --url=https://... --token=...
```

## JSON Envelope Format

All commands output a JSON envelope in non-TTY mode (pipes, CI, subprocesses). Force JSON with `--json`, force human-readable with `--text`.

```json
{
  "command": "deploy",
  "version": "1.3.0",
  "exitCode": 0,
  "result": { },
  "logs": [
    { "level": "info", "ts": "2026-03-05T09:30:00Z", "message": "Starting deployment" }
  ],
  "errors": []
}
```

### Per-Command Result Shapes

**validate**
```json
{ "valid": true, "errors": [] }
{ "valid": false, "errors": [{ "path": "channels.0.currencyCode", "message": "Required" }] }
```

**diff**
```json
{
  "summary": { "totalChanges": 3, "creates": 1, "updates": 1, "deletes": 1 },
  "operations": [
    { "operation": "CREATE", "entityType": "Categories", "entityName": "electronics", "changes": [] }
  ],
  "hasDestructiveOperations": true
}
```

**deploy --plan**
```json
{
  "status": "plan",
  "summary": { "creates": 1, "updates": 0, "deletes": 0, "noChange": 5 },
  "operations": [],
  "willDeleteEntities": false,
  "configFile": "config.yml",
  "saleorUrl": "https://store.saleor.cloud/graphql/"
}
```

**deploy** (executed)
```json
{
  "status": "completed",
  "summary": { "succeeded": 6, "failed": 0, "skipped": 0 },
  "operations": [],
  "duration": "12.3s",
  "reportPath": ".configurator/reports/deploy/store-abc_2026-03-05_09h30m00s.json"
}
```

**introspect**
```json
{
  "status": "success",
  "configPath": "config.yml"
}
```

### Parsing Examples

```bash
# Parse exit code and result
OUTPUT=$(pnpm dlx @saleor/configurator validate --json 2>/dev/null)
EXIT_CODE=$(echo "$OUTPUT" | jq -r '.exitCode')
IS_VALID=$(echo "$OUTPUT" | jq -r '.result.valid')

# Extract errors
echo "$OUTPUT" | jq -r '.result.errors[] | "\(.path): \(.message)"'

# Check for destructive operations in diff
DESTRUCTIVE=$(pnpm dlx @saleor/configurator diff --json 2>/dev/null | jq -r '.result.hasDestructiveOperations')
```

```javascript
// Node.js / agent parsing
const { execSync } = require("child_process");
const output = execSync("pnpm dlx @saleor/configurator diff --json", {
  encoding: "utf-8",
  stdio: ["pipe", "pipe", "pipe"],
});
const envelope = JSON.parse(output);

if (envelope.exitCode !== 0) {
  console.error("Errors:", envelope.errors);
}
```

## Optimal Workflow Sequence

```
validate --json
    |
    v
  valid? --no--> fix config.yml --> validate again
    |
   yes
    |
    v
diff --json
    |
    v
  changes? --no--> done (in sync)
    |
   yes
    |
    v
deploy --plan --json
    |
    v
  review plan
    |
    v
deploy --json
    |
    v
  exitCode 0? --no--> check errors, drill down
    |
   yes
    |
    v
  done
```

## Entity-Scoped Drill-Down

When a deployment fails partially (exit code 5), drill into specific entities:

```bash
# 1. Deploy fails with partial errors
pnpm dlx @saleor/configurator deploy --json 2>/dev/null
# exitCode: 5, errors mention "Categories/electronics"

# 2. Drill into that entity type
pnpm dlx @saleor/configurator diff --entity-type "Categories" --json 2>/dev/null

# 3. Drill into the specific entity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saleor/configurator](https://github.com/saleor/configurator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
