---
trigger: always_on
description: This document explains how to build **external provider adapters** for OpsOrch Core.
---

# OpsOrch Adapter Development Guide

This document explains how to build **external provider adapters** for OpsOrch Core.

OpsOrch Core is a **stateless orchestration layer**.  
It provides unified APIs, routing, secret management, and schema boundaries — but **does not define or enforce the exact shape of incidents, logs, metrics, tickets, or messages**.

Those schemas will evolve during implementation.  
Adapters must **conform to whatever schema is currently defined**, but this guide will not prescribe their structure.

---

## 1. Architecture Overview

OpsOrch Core delegates all provider-specific logic to external adapters.

```
opsorch-core/
  api/
  registry/
  schema/      <-- evolving, not finalized
  secret/
  runtime/

opsorch-adapter-<provider>/
  incident/
  alert/
  log/
  metric/
  ticket/
  messaging/
```

- **opsorch-core**: routing, secret management, registry, HTTP layer.
- **adapter repos**: implement the capability interfaces.

Adapters do not live inside opsorch-core. They are loaded at runtime either via registered constructors or local plugin binaries.

---

## 2. Capability Interfaces (Shape Agnostic)

OpsOrch defines **interfaces only**, not the schema contents.

Example (simplified):

```go
type IncidentProvider interface {
    Query(ctx context.Context, query schema.IncidentQuery) ([]schema.Incident, error)
    Get(ctx context.Context, id string) (schema.Incident, error)
    Create(ctx context.Context, in schema.CreateIncidentInput) (schema.Incident, error)
    Update(ctx context.Context, id string, in schema.UpdateIncidentInput) (schema.Incident, error)

    GetTimeline(ctx context.Context, id string) ([]schema.TimelineEntry, error)
    AppendTimeline(ctx context.Context, id string, entry schema.TimelineAppendInput) error
}
```

**Important:**  
The *contents* of `schema.Incident`, `schema.TimelineEntry`, etc. are intentionally not fixed here.  
They will be defined as we implement the system.

**Adapters must align with whatever schema version opsorch-core currently exposes.**

---

## 3. Adapter Structure

Each adapter:

- lives in its own repo  
- implements one or more capability interfaces  
- exposes a constructor, usually named `New(config map[string]any)`  
- receives decrypted config from opsorch-core  
- returns normalized objects matching current schemas  
- never stores state

Example skeleton:

```go
func New(config map[string]any) (incident.Provider, error) {
    token := config["apiKey"].(string)
    base := config["baseUrl"].(string)

    return &MyProvider{Token: token, BaseURL: base}, nil
}
```

The provider shape is up to you.

---

## 4. Config + Secrets

OpsOrch Core handles:

- storage
- encryption
- decryption
- validation

Adapters receive **only a decrypted config map**, never raw secrets or tokens.

You **must not log**, print, or expose config values.

---

## 5. Registration

Adapters register themselves in external repos.

OpsOrch Core uses a registry to match capability + provider:

```go
incident.RegisterProvider("pagerduty", pagerduty.New)
incident.RegisterProvider("incidentio", incidentio.New)
alert.RegisterProvider("prometheus", prometheus.New)
```

Providers can be:

- built-in for OSS
- dynamically loaded
- injected through config

### Plugin-Based Loading (no remote services)

OpsOrch Core can launch a local adapter binary as a child process (no network hops) when `OPSORCH_<CAP>_PLUGIN` is set or the provider config includes a `plugin` path. RPC is JSON over stdin/stdout:

- Request: `{ "method": "<capability>.<operation>", "config": {...}, "payload": {...} }`
- Response: `{ "result": <value>, "error": "<msg>" }`

**RPC Methods by Capability:**

| Capability | Method | Payload |
|------------|--------|---------|
| incident | `incident.query` | `IncidentQuery` |
| incident | `incident.list` | `null` |
| incident | `incident.get` | `{ "id": string }` |
| incident | `incident.create` | `CreateIncidentInput` |
| incident | `incident.update` | `{ "id": string, "input": UpdateIncidentInput }` |
| incident | `incident.timeline.get` | `{ "id": string }` |
| incident | `incident.timeline.append` | `{ "id": string, "entry": TimelineAppendInput }` |
| alert | `alert.query` | `AlertQuery` |
| alert | `alert.get` | `{ "id": string }` |
| log | `log.query` | `LogQuery` |
| metric | `metric.query` | `MetricQuery` |
| metric | `metric.describe` | `QueryScope` |
| ticket | `ticket.query` | `TicketQuery` |
| ticket | `ticket.get` | `{ "id": string }` |
| ticket | `ticket.create` | `CreateTicketInput` |
| ticket | `ticket.update` | `{ "id": string, "input": UpdateTicketInput }` |
| messaging | `messaging.send` | `Message` |
| service | `service.query` | `ServiceQuery` |
| deployment | `deployment.query` | `DeploymentQuery` |
| deployment | `deployment.get` | `{ "id": string }` |
| team | `team.query` | `TeamQuery` |
| team | `team.get` | `{ "id": string }` |
| team | `team.members` | `{ "teamID": string }` |
| orchestration | `orchestration.plans.query` | `OrchestrationPlanQuery` |
| orchestration | `orchestration.plans.get` | `{ "planId": string }` |
| orchestration | `orchestration.runs.query` | `OrchestrationRunQuery` |
| orchestration | `orchestration.runs.get` | `{ "runId": string }` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpsOrch/opsorch-core](https://github.com/OpsOrch/opsorch-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
