---
trigger: always_on
description: This file states the **principles** for working in this codebase. It is not a how-to. Code details (directory layout, tech-stack versions, function names, step-by-step recipes) belong in the code itself — read the code when you need them.
---

# CLAUDE.md - AI Assistant Guide for Apache SkyWalking Horizon UI

This file states the **principles** for working in this codebase. It is not a how-to. Code details (directory layout, tech-stack versions, function names, step-by-step recipes) belong in the code itself — read the code when you need them.

## What this project is

**Horizon UI** is the next-generation web UI for [Apache SkyWalking](https://github.com/apache/skywalking). The goal is **feature parity** with [skywalking-booster-ui](https://github.com/apache/skywalking-booster-ui) on the **same OAP GraphQL query-protocol and MQE**, with a modernized, dense, dark-first design. This is a **greenfield rewrite**, not a fork. Backend APIs do not change.

## How to work (the only workflow that matters)

> **Correctness comes first. Speed comes from getting it right the first time, not from skipping steps.**

Every non-trivial change follows the same loop:

1. **Read the code.** Read it end-to-end along the path you intend to change: call site → handler → response shape → consumer. Comments, this file, and prior conversation are *reference* — the source of truth is the current code on disk.
2. **Implement.** Match what the surrounding code already does. If you have to invent a pattern, you probably haven't finished step 1.
3. **Validate against a live OAP.** Run the change against a real OAP server and confirm the wire request/response and the rendered UI. Type-checks and unit tests verify code, not feature behavior.

If no OAP is available to validate against, **stop and ask the developer to provide one** (URL, credentials, demo endpoint, port-forward — whatever). Do not guess at wire shapes, do not mock the data and call it done, do not declare the work complete from a green type-check alone. "I couldn't validate" is an honest, acceptable outcome — silently shipping unvalidated changes is not.

When you can't reproduce the user's symptom locally, say so. Don't invent a fix.

## Backend compatibility

The UI talks to OAP through the **GraphQL query-protocol** (same as booster-ui) and through OAP's admin REST surface. Both contracts are **fixed** — owned by the skywalking repo, not this one.

- **Do not invent fields.** If a screen needs data the protocol doesn't expose, flag it. The right fix is a query-protocol change upstream, not a UI hack or a BFF-side fabrication.
- **The schemas and Java implementations are the authoritative spec** — read them (`oap-server/server-query-plugin/.../query-protocol/*.graphqls` and `oap-server/server-core/.../query/`) before guessing at a wire shape. Stand up a local OAP (the SkyWalking repo ships a docker-compose) for smoke-testing wire changes.
- **Booster-ui is the working reference.** When in doubt about how a query is shaped or paged, look at how booster-ui does it.

### Metric entity-scope is load-bearing

Every OAP metric lives under exactly one entity scope (Service / ServiceInstance / Endpoint / relations / Process / All). OAP does not auto-rollup between scopes — querying at the wrong scope returns empty results regardless of MQE wrapping. Before adding or moving a metric, verify its scope against the OAP catalog and confirm it matches the page that will render it. Never invent a BFF-side rollup to bridge a scope mismatch — move the metric or leave the slot empty.

### Time, step, and timezone

- **Step precision is page-family-specific.** Dashboards / overviews / landing scale step with the rolling window (MINUTE / HOUR / DAY). Alarms / traces / logs / live debugger use SECOND because they query event-style data anchored at second precision — MINUTE rounding chops off the most recent (most interesting) events. MQE traffic backdrops use MINUTE because metrics are aggregated at minute granularity.
- **String format is determined by step.** Mixing them throws `verifyDateTimeString` on OAP. Read `DurationUtils.java` in the skywalking repo for the canonical mapping.
- **OAP has a per-request bucket cap.** Long windows must be chunked. Storage backends impose stricter caps that vary by backend — probe, don't assume.
- **All time strings are OAP-server local.** Not UTC, not browser-local. The server's offset is exposed via `getTimeInfo`. The BFF owns this conversion; the UI displays in browser-local (echarts handles ms → local natively).
- **Per-page vs. global time.** The topbar time picker applies only to layer dashboards + overviews. Triage / investigation pages (alarms, traces, logs, profilings, live debugger) own their own time range — do not subscribe them to the global ticker.
- **Picker wiring is a two-sided contract.** The time range only reaches OAP when the UI composable forwards it AND the BFF route accepts it. Verify the actual request, not the intent.

### Other OAP sharp edges

- **Storage backends have undocumented limits.** Page sizes, nested selections, and per-record sub-queries fail at backend-specific thresholds. Degrade list queries to the cheapest selection that satisfies the screen; probe before defaulting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/skywalking-horizon-ui](https://github.com/apache/skywalking-horizon-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
