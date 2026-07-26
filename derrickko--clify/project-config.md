---
trigger: always_on
description: Rules and contracts for generated CLI repos. Rigid contracts (error format, `.clify.json` shape) must be followed exactly. Flexible guidance (command structure, nesting) should be adapted to the API.
---

# clify Conventions

Rules and contracts for generated CLI repos. Rigid contracts (error format, `.clify.json` shape) must be followed exactly. Flexible guidance (command structure, nesting) should be adapted to the API.

---

## CLI Command Structure

Generated CLIs use the resource-action pattern:

```
<api>-cli <resource> <action> [flags]
```

### Standard Actions

Map HTTP methods to actions:

| HTTP Method | Action | Notes |
|-------------|--------|-------|
| GET (collection) | `list` | Returns array |
| GET (single) | `get` | Requires `--id` |
| POST (create) | `create` | |
| PUT/PATCH | `update` | Requires `--id` |
| DELETE | `delete` | Requires `--id` |

### Non-CRUD Actions

When an endpoint doesn't map to standard CRUD, use the API's own verb:

```
stripe-cli charges capture --id ch_xxx
github-cli repos merge-upstream --repo owner/name
twilio-cli messages send --to +1234567890 --from +0987654321
```

Don't force non-CRUD actions into CRUD semantics. If the API calls it "verify", the action is `verify`.

### Nesting Depth

Cap at two levels: `<resource> <action>` or `<resource> <sub-resource> <action>`.

Flatten anything deeper with flags:

```
# Good (2 levels):
github-cli repos pulls list --repo owner/name

# Bad (3 levels):
github-cli repos pulls comments list --repo owner/name

# Good (flattened):
github-cli pull-comments list --repo owner/name --pull 42
```

### Resource Naming

- Use the API's own terminology (kebab-case if multi-word)
- Don't rename to camelCase or PascalCase
- Hyphenated names are fine: `api-keys`, `pull-requests`

---

## Global Flags

Every command supports these flags. They are parsed before resource routing.

| Flag | Type | Default | Behavior |
|------|------|---------|----------|
| `--json` | boolean | true when piped | Structured JSON output |
| `--dry-run` | boolean | false | Show request without executing |
| `--help`, `-h` | boolean | false | Show usage |
| `--version`, `-v` | boolean | false | Print version |
| `--verbose` | boolean | false | Include request/response headers |
| `--all` | boolean | false | Auto-paginate (fetch all pages) |

### Global Flag Parsing

Global flags must be separated from per-command flags before parsing. Use a known-set filter — do NOT use `parseArgs` with `strict: false` for global parsing, as it consumes unknown flags as booleans and breaks per-command string flags.

```js
const GLOBAL_FLAGS = new Set(["--json", "--dry-run", "--help", "-h", "--version", "-v", "--verbose", "--all"]);
const globalArgv = [];
const remainingArgv = [];
for (const arg of process.argv.slice(2)) {
  if (GLOBAL_FLAGS.has(arg)) globalArgv.push(arg);
  else remainingArgv.push(arg);
}
```

---

## Per-Command Flags

Generated from the API's documented parameters:

| Parameter Type | Flag Style |
|---------------|------------|
| Path params | `--id`, `--repo` (required) |
| Query params | Optional flags |
| Body fields | Individual flags |
| Raw JSON body | `--body` escape hatch |

### Special Flags

| Flag | When | Behavior |
|------|------|----------|
| `--all` | List endpoints with pagination | Auto-paginate |
| `--file <path>` | Upload endpoints | Read file, use FormData |
| `--output <path>` | Binary response endpoints | Write response to file |
| `--body <json>` | Any mutating endpoint | Raw JSON body (overrides individual flags) |

### Pagination

Return one page by default. Include `next_cursor` in JSON output when more pages exist. `--all` fetches every page and returns the combined result.

---

## Structured Error Output

When `--json` is active (or stdout is piped), errors emit:

```json
{
  "type": "error",
  "code": "rate_limited",
  "message": "Rate limited. Retry after 30s.",
  "retryable": true,
  "retryAfter": 30
}
```

### Error Taxonomy

| Code | Retryable | HTTP Status | Meaning |
|------|-----------|-------------|---------|
| `auth_missing` | no | — | No API key in `.env` |
| `auth_invalid` | no | 401 | Key rejected |
| `validation_error` | no | 400, 422 | Bad request |
| `not_found` | no | 404 | Resource doesn't exist |
| `forbidden` | no | 403 | Insufficient permissions |
| `conflict` | no | 409 | State conflict |
| `rate_limited` | yes | 429 | Too many requests |
| `server_error` | yes | 5xx | API server error |
| `network_error` | yes | — | Connection failed/timeout |
| `timeout` | yes | — | Request exceeded timeout |

### Rules

- `retryAfter` is optional on ALL retryable errors (not just `rate_limited`)
- Parse `Retry-After` header when present on any retryable error
- CLI never retries — retry logic lives in the SKILL.md wrapper (agent decides)
- Human-readable format uses `console.error` (stderr), not stdout
- Exit code is always 1 for errors

---

## `.clify.json` Shape

Every generated repo has this at the root:

```json
{
  "apiName": "github",
  "docsUrl": "https://docs.github.com/en/rest",
  "crawledUrls": ["https://docs.github.com/en/rest/repos", "..."],
  "contentHash": "sha256:abc123...",
  "generatedAt": "2026-04-06T12:00:00Z",
  "clifyVersion": "0.1.0",
  "nodeMinVersion": "20"
}
```

| Field | Type | Description |
|-------|------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derrickko/clify](https://github.com/derrickko/clify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
