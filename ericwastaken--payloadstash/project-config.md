---
trigger: always_on
description: Use this skill when asked to create, edit, or extend a PayloadStash YAML config file.
---

# Skill: Write a PayloadStash Config File

Use this skill when asked to create, edit, or extend a PayloadStash YAML config file.

---

## What PayloadStash Does

PayloadStash reads a YAML config, executes HTTP requests (sequentially or concurrently), writes each response to disk, and optionally asserts on response values. Every run produces a resolved config, a run log, a results CSV, and a markdown report.

---

## File Structure

```yaml
# Optional: YAML anchors for reuse
my_headers: &my_headers
  Content-Type: application/json

# Optional: dynamic value generators
dynamics:
  patterns:
    myId:
      template: "ID-${hex:8}"
  sets:
    envs: ["stage", "prod"]

# Required: the config body
StashConfig:
  Name: MyRun                  # required, unique run name
  Defaults: ...                # required
  Forced: ...                  # optional
  Sequences: [...]             # required, non-empty list
```

Top-level keys outside `StashConfig` and `dynamics` are ignored by the parser — use them freely for YAML anchors.

---

## Defaults (required)

```yaml
Defaults:
  URLRoot: https://api.example.com   # required, no trailing slash
  FlowControl:                        # required
    DelaySeconds: 0                   # int >= 0
    TimeoutSeconds: 30                # int >= 0
  InsecureTLS: false                  # optional; skips TLS verification
  Headers:                            # optional
    Content-Type: application/json
  Body:                               # optional
    commonField: value
  Query:                              # optional
    version: v2
  Retry:                              # optional; see Retry section
    Attempts: 3
    BackoffStrategy: exponential
    BackoffSeconds: 0.5
  Response:                           # optional
    PrettyPrint: true
    Sort: false
```

---

## Forced (optional)

Keys in `Forced` are overlaid on top of Defaults and per-request values last. Use it to inject values that must always win (e.g., an auth header that cannot be overridden by individual requests).

```yaml
Forced:
  Headers:
    Authorization: { $secrets: TOKEN }
  Body:
    tenantId: "acme"
  Query: ...
  Retry: ...
```

---

## Sequences

Each sequence is a named group of requests that run either sequentially or concurrently.

```yaml
Sequences:
  - Name: MySequence          # required; unique across all sequences
    Type: Sequential          # Sequential | Concurrent
    # ConcurrencyLimit: 4     # required when Type=Concurrent; forbidden when Sequential
    Requests:
      - RequestKey:           # unique within this sequence; used in filenames and reports
          Method: POST        # GET | POST | PUT | PATCH | DELETE | HEAD | OPTIONS
          URLPath: /v1/thing  # appended to URLRoot
          Headers: ...        # optional; overrides Defaults.Headers
          Body: ...           # optional; overrides Defaults.Body
          Query: ...          # optional; overrides Defaults.Query
          FlowControl: ...    # optional; overrides Defaults.FlowControl fields
          Retry: ...          # optional; set to Null to disable retries for this request
          Response: ...       # optional; overrides Defaults.Response
          InsecureTLS: false  # optional; overrides Defaults.InsecureTLS
          dynamics: ...       # optional; request-level patterns merged with top-level
          Capture: ...        # optional; extract values from the response
          Expect: ...         # optional; assert on the response
```

**Merge rules for Headers / Body / Query:**
- Start with the request-level value if present; otherwise use Defaults.
- Overlay Forced on top last.

**Retry precedence:**
- `request.Retry` (even if `Null`) beats `Defaults.Retry`.
- Only falls through to Defaults when the request omits `Retry` entirely.

---

## Retry

```yaml
Retry:
  Attempts: 3                         # int >= 1 (total tries including first)
  BackoffStrategy: exponential        # fixed | exponential
  BackoffSeconds: 0.5                 # float >= 0; base delay
  Multiplier: 2.0                     # float > 0; only for exponential (default 2.0)
  MaxBackoffSeconds: 10.0             # float >= 0; cap per-retry delay
  MaxElapsedSeconds: 60.0             # float >= 0; total budget across all retries
  Jitter: true                        # bool or "min" | "max"
  RetryOnStatus: [429, 500, 502, 503, 504]
  RetryOnNetworkErrors: true
  RetryOnTimeouts: true
```

Disable retries for a specific request with `Retry: Null`.

---

## Response Formatting

```yaml
Response:
  PrettyPrint: true   # pretty-print JSON (via Rich) and XML before writing to file
  Sort: true          # sort JSON keys / XML elements; implies PrettyPrint
```

---

## Special Operators

### `$dynamic` — named pattern from the dynamics section

```yaml
# resolve-time (default): same value used everywhere the pattern appears in this run
Body:
  id: { $dynamic: myId }

# request-time: fresh value generated right before each HTTP call
Body:
  id: { $dynamic: myId, when: request }
```

Requires a `dynamics.patterns.<name>.template` entry at the top of the file (or in the request's own `dynamics` block).

### `$pattern` — inline request-time template


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericwastaken/PayloadStash](https://github.com/ericwastaken/PayloadStash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
