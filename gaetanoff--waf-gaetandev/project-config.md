---
trigger: always_on
description: Spec-driven observability — structured logging, metrics, tracing, SLOs, alerting defined as contracts
---


# Observability (Spec-Driven)

## Overview

Observability is not an afterthought — it's a **contract**. Define logging, metrics, and tracing specs alongside your API and data contracts. Every observable signal traces back to a spec.

## Structured Logging Contract

### Log Entry Schema

Define a standard log format as a JSON Schema:

```json
{
  "title": "LogEntry",
  "type": "object",
  "required": ["timestamp", "level", "message", "service"],
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp"
    },
    "level": {
      "type": "string",
      "enum": ["debug", "info", "warn", "error", "fatal"]
    },
    "message": {
      "type": "string",
      "description": "Human-readable log message"
    },
    "service": {
      "type": "string",
      "description": "Service name emitting the log"
    },
    "requestId": {
      "type": "string",
      "format": "uuid",
      "description": "Correlation ID — traces a request across services"
    },
    "traceId": {
      "type": "string",
      "description": "Distributed trace ID (W3C Trace Context)"
    },
    "spanId": {
      "type": "string",
      "description": "Current span ID"
    },
    "userId": {
      "type": "string",
      "description": "Authenticated user ID (never log PII)"
    },
    "context": {
      "type": "object",
      "description": "Additional structured context (endpoint, method, duration, etc.)"
    },
    "error": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "message": { "type": "string" },
        "stack": { "type": "string" }
      }
    }
  }
}
```

### Logging Rules

- **Always log**: request start/end, errors, auth failures, business events, performance anomalies.
- **Never log**: passwords, tokens, PII (email, phone, address), credit card numbers, API keys.
- **Log levels**:
  - `debug`: detailed diagnostic info (disabled in production).
  - `info`: normal operations (request handled, job completed, user action).
  - `warn`: unexpected but recoverable situations (deprecated API call, retry, degraded service).
  - `error`: failures that need attention (unhandled exception, external service down).
  - `fatal`: system cannot continue (missing critical config, database unreachable on startup).
- Every log entry includes `requestId` for correlation.
- Use structured logging (JSON) — never `console.log("user: " + user)`.

## Metrics Contract

### RED Method (Request-Driven Services)

Define these metrics for every API endpoint in the spec:

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `http_requests_total` | Counter | Total requests received | `method`, `path`, `status` |
| `http_request_duration_seconds` | Histogram | Request latency | `method`, `path` |
| `http_request_errors_total` | Counter | Total error responses (4xx, 5xx) | `method`, `path`, `status` |

### USE Method (Resource-Driven Systems)

| Metric | Type | Description |
|--------|------|-------------|
| `cpu_utilization_ratio` | Gauge | CPU usage (0-1) |
| `memory_utilization_bytes` | Gauge | Memory usage |
| `db_connections_active` | Gauge | Active database connections |
| `db_connections_pool_size` | Gauge | Connection pool size |
| `queue_depth` | Gauge | Number of messages waiting in queue |
| `disk_utilization_ratio` | Gauge | Disk usage (0-1) |

### Business Metrics (From Specs)

Derive business metrics from behavior specs:

```yaml
# Example: derived from the checkout behavior spec
business_metrics:
  - name: orders_created_total
    type: counter
    description: Total orders created
    labels: [payment_method, currency]
    spec_ref: specs/features/checkout.feature

  - name: order_value_total
    type: counter
    description: Total order value in cents
    labels: [currency]

  - name: checkout_abandonment_total
    type: counter
    description: Carts abandoned during checkout
    labels: [step]
```

## Distributed Tracing Contract

### Span Naming Convention

Derive span names from API contracts:

| Operation | Span Name |
|-----------|-----------|
| HTTP endpoint | `HTTP {METHOD} {path}` (e.g. `HTTP GET /api/v1/users`) |
| Database query | `DB {operation} {table}` (e.g. `DB SELECT users`) |
| External API call | `EXT {service} {operation}` (e.g. `EXT Stripe createCharge`) |
| Message publish | `MSG PUBLISH {channel}` (e.g. `MSG PUBLISH orders.created`) |
| Message consume | `MSG CONSUME {channel}` |
| Background job | `JOB {name}` (e.g. `JOB sendWelcomeEmail`) |

### Required Span Attributes

```yaml
# From OpenAPI spec
http.method: GET
http.url: /api/v1/users
http.status_code: 200
http.request_id: <uuid>

# From data contracts
db.system: postgresql
db.statement: SELECT * FROM users WHERE id = $1
db.operation: SELECT

# From error contracts
error: true
error.code: NOT_FOUND
error.message: User not found
```

### Trace Context Propagation

- Use **W3C Trace Context** (`traceparent`, `tracestate` headers) for inter-service propagation.
- Every HTTP client must forward trace headers.
- Every message must include trace context in metadata.
- Every background job must inherit the parent trace.

## SLO/SLI/SLA Definitions

### SLO Spec Format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
