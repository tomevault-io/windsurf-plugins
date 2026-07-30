---
trigger: always_on
description: How to add telemetry so every instrumentation reads the same way. The framework
---

# Telemetry conventions

How to add telemetry so every instrumentation reads the same way. The framework
lives in `src/telemetry`; the per-domain instrumentation business code talks to
lives here in `src/instrumentation`.

## Checklist

- One instrumentation class per domain (`FooTelemetry`) wrapping
  `TelemetryService`; business code imports that, never a raw span.
- Event name is `domain.snake_case`; point-in-time logs use past tense.
- Event names and attribute keys follow OTel: lowercase, `.` for hierarchy, `_`
  to split words, never camelCase. Enumerated values are typed `snake_case`
  unions, never bare `string`.
- Numbers go in `measurements` (raw), never pre-bucketed into string properties.
- Set attributes imperatively with `setProperty`/`setMeasurement`; never add a
  return value that exists only to be logged.
- No secrets, tokens, query strings, file paths, or other unbounded user
  content in properties; routes go through `normalizeRoute`.
- Let the framework set `result`; add a domain `outcome` only when an operation
  has several success modes. Errors go to a typed `error.type` union; non-error
  early exits call `markAborted`.

## Layers

- **Framework** (`src/telemetry`): `TelemetryService` (`trace`/`log`/`logError`)
  hands out `Span` handles and owns IDs, timing, `result`, level-gating, and the
  wire format. Telemetry-off is handled here (`NOOP_SPAN`), so instrumentation
  never checks whether telemetry is enabled.
- **Instrumentation** (`src/instrumentation/*`): one typed class per domain, the
  only telemetry surface business code sees.

## Structure

- Split instrumentation files along the same boundaries as the business code,
  not one catch-all module.
- Shared span helpers (`recordError`, `recordAborted`) live in one shared module,
  not duplicated per file.
- Record-error-then-rethrow-outside-the-span logic lives once per class, in a
  single private helper, not in every `traceX` method.

## Threading

Spans are passed **explicitly** as a callback argument; there is no
ambient/active-span context. Two patterns keep telemetry out of business logic:

1. **Imperative attributes** — `span.setProperty("outcome", "cache_hit")` at the
   point the value is known. This is the standard OpenTelemetry model.
2. **Typed phases** — wrap an async step in `span.phase(...)` and read one
   property off its _natural_ return value, e.g.
   `trace.versionCheck(() => this.checkBinary(...))`. Extraction stays out of
   the business function.

Never return a value purely so a caller can log it; that couples the return type
to observability. Returning is fine when the business uses the value too.

## Callers

- Declare telemetry dimensions explicitly at the call site; pass `source: "uri"`
  rather than inferring it from which arguments happen to be set.
- Keep business bodies in named private `runX(args, trace)` methods; the public
  method just opens the span and wraps them. Small diffs, named telemetry seam.
- When sibling events share a correlating property, emit it on every event in the
  family; don't drop it from new ones.

## Spans, phases, logs

- `trace(name, fn, props?, meas?)` — a span with framework-set `result` and
  `durationMs`. Use for an operation with a start and end.
- `span.phase(name, fn, ...)` — the same, nested (composed as `parent.child`);
  child names cannot contain `.`.
- `span.log(name, ...)` / `span.logError(name, error, ...)` — point-in-time
  events under a span, no `result`/`durationMs`. Use for discrete signals.

## Naming

| Thing                      | Convention                                  | Examples                                                  |
| -------------------------- | ------------------------------------------- | --------------------------------------------------------- |
| Event name                 | `domain.snake_case`                         | `cli.resolve`, `remote.setup`, `connection.dropped`       |
| Point-in-time log          | past tense                                  | `connection.dropped`, `ssh.process.lost`                  |
| Child phase                | bare `snake_case`                           | `cache_lookup`, `version_check`, `connection_handoff`     |
| Property / measurement key | lowercase; `_` splits words, `.` for groups | `cache_source`, `error.type`, `status.from`               |
| Enumerated value           | typed `snake_case` union                    | `"cache_hit"`, `"session_token"`, `"unrecoverable_close"` |

This is the [OTel attribute convention](https://opentelemetry.io/docs/specs/semconv/general/naming/):
`.` is the namespace delimiter, `_` joins words within a segment, never
camelCase. Default to a flat `snake_case` key; use `.` only to group genuinely
related attributes (a `status.from` / `status.to` pair). Keep phase names
subject-first within a domain (`agent_resolve`, not `resolve_agent`).

**Methods.** Span-wrapper methods are `trace<Noun>` (`traceOpen`,
`traceConfirmationPrompt`); don't echo the event's past-tense suffix
(`traceUpdateConfirmationPrompted`), and drop qualifiers the class implies.

**Grouping.** Group related events under a shared dotted namespace so a prefix
query returns the whole family: `workspace.update.triggered` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/vscode-coder](https://github.com/coder/vscode-coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
