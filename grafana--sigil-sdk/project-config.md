---
trigger: always_on
description: Guide Cursor agents to add Grafana Sigil instrumentation to consumer codebases.
---


# Cursor Prompt: Sigil Instrumentation

You are running in Cursor with full repository context.
- Keep edits surgical and grouped by opportunity.
- Run focused checks after each opportunity and include results.

## Sigil Agent-First Instrumentation Brief

You are acting as a coding agent inside this repository. Your goal is to add or improve Grafana Sigil instrumentation with minimal, safe changes.

## Mission

1. Find AI generation and tool/agent execution paths.
2. Add Sigil instrumentation using the local language SDK where possible.
3. Preserve behavior and keep diffs small.
4. Add or update tests for changed instrumentation behavior.
5. Explain what was instrumented and why.

## Output contract (required)

Return:

- Top opportunities first (highest traffic / highest impact)
- For each opportunity:
  - exact file path(s)
  - why this location matters
  - concrete diff proposal
  - test plan
  - any risk or compatibility concern

## Sigil architecture and ingest model (must follow)

- Sigil uses generation-first ingest:
  - gRPC: `sigil.v1.GenerationIngestService.ExportGenerations`
  - HTTP parity: `POST /api/v1/generations:export`
- Traces/metrics go through OTEL collector/alloy, not through Sigil ingest.
- Required generation modes:
  - non-stream: `SYNC`
  - stream: `STREAM`
- Raw provider artifacts are default OFF and only enabled for explicit debug opt-in.

## OTel setup (required)

The Sigil SDK internally emits OTel spans and metrics (`gen_ai.client.operation.duration`, `gen_ai.client.token.usage`, `gen_ai.client.time_to_first_token`, `gen_ai.client.tool_calls_per_operation`). **Without a configured TracerProvider and MeterProvider these go to the default no-op and are silently lost.**

The SDK does NOT create OTel providers — that is the application's responsibility. Always ensure the app configures providers BEFORE creating the Sigil client, and shuts them down AFTER `sigil.shutdown()`.

Traces and metrics can be sent to Grafana Cloud in two ways. Always use env vars (`OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_HEADERS`) so the app doesn't hardcode assumptions.

### Option A — Direct to Grafana Cloud (no collector needed)

Send OTLP straight to the Grafana Cloud OTLP gateway. The exact URL is stack-specific — get it from the **Grafana Cloud portal → stack Details page** ([docs](https://grafana.com/docs/grafana-cloud/send-data/otlp/send-data-otlp)). Authentication uses Basic auth with instance ID and a Cloud API token.

Env vars:
```
OTEL_EXPORTER_OTLP_ENDPOINT=https://<your-otlp-gateway-url>   # from Cloud portal
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic <base64(instance_id:cloud_api_token)>
```

The OTel SDK exporters read these env vars automatically — no extra code needed beyond the provider setup below.

### Option B — Via Alloy / OTel Collector (optional)

Run a local Alloy or OTel Collector that receives unauthenticated OTLP and forwards to Cloud with credentials. Useful for centralized token management, retries, relabeling, and metadata enrichment. Common local ports: 4318 (OTLP/HTTP), 4317 (OTLP/gRPC).

Env vars:
```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

### Provider setup (required for both options)

The snippets below configure TracerProvider and MeterProvider using OTLP/HTTP exporters that honour the env vars above.

#### Python
```python
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.sdk.resources import Resource
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.exporter.otlp.proto.http.metric_exporter import OTLPMetricExporter

resource = Resource.create({"service.name": "my-app"})

tp = TracerProvider(resource=resource)
tp.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(tp)

mp = MeterProvider(resource=resource, metric_readers=[
    PeriodicExportingMetricReader(OTLPMetricExporter())
])
metrics.set_meter_provider(mp)
# Deps: opentelemetry-sdk, opentelemetry-exporter-otlp-proto-http
```

#### Go
```go
traceExp, _ := otlptracehttp.New(ctx)
tp := sdktrace.NewTracerProvider(sdktrace.WithBatcher(traceExp), sdktrace.WithResource(res))
otel.SetTracerProvider(tp)
defer tp.Shutdown(ctx)

metricExp, _ := otlpmetrichttp.New(ctx)
mp := sdkmetric.NewMeterProvider(sdkmetric.WithReader(sdkmetric.NewPeriodicReader(metricExp)), sdkmetric.WithResource(res))
otel.SetMeterProvider(mp)
defer mp.Shutdown(ctx)
```

#### JS/TS
```typescript
import { metrics } from '@opentelemetry/api';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { MeterProvider, PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';

const tp = new NodeTracerProvider({ resource });
tp.addSpanProcessor(new BatchSpanProcessor(new OTLPTraceExporter()));
tp.register();


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/sigil-sdk](https://github.com/grafana/sigil-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
