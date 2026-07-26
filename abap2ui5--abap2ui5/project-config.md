---
trigger: always_on
description: > This file follows the cross-tool AGENTS.md convention and is the single
---

# AGENTS.md — AI Assistant Guide for abap2UI5

> This file follows the cross-tool AGENTS.md convention and is the single
> agent instruction file of this repository — there is no separate
> `CLAUDE.md`; Claude Code reads `AGENTS.md` natively.

## Project Overview

abap2UI5 is a framework for building SAP UI5 applications purely in ABAP — no JavaScript, OData, or RAP required. It supports all ABAP releases from NW 7.02 to ABAP Cloud, running in both on-premise and cloud environments.

**Current version:** 1.142.0 (defined in `src/02/z2ui5_if_app.intf.abap`)
**License:** MIT
**Homepage:** https://abap2UI5.org
**Language:** English — all code, comments, commit messages, PRs, issues, documentation, and communication must be in English.

### Related Repositories

| Repository | Purpose |
|---|---|
| [abap2UI5](https://github.com/abap2UI5/abap2UI5) | Core framework (this repo) |
| [samples](https://github.com/abap2UI5/samples) | Sample applications and usage examples |
| [docs](https://github.com/abap2UI5/docs) | Project documentation |
| [abap-util](https://github.com/abap-util/abap-util) | Master catalog of the platform utilities (all classes, all methods) — `src/00/03/` holds renamed copies of the classes the framework needs, with the context class trimmed to the used methods (see "Vendored utility classes" below) |

> **Building apps?** This file is the briefing for AI assistants working **on the framework itself**. For everything an AI needs to **build apps with** abap2UI5 — app template, client API, view-building patterns, lifecycle, deprecated controls — see the single canonical guide at <https://abap2ui5.github.io/docs/advanced/agent.html>.

## Architecture

### How It Works — The Roundtrip

abap2UI5 is a **stateful SPA** (Single Page Application). The browser loads a UI5 shell once via HTTP GET, then communicates with the ABAP backend exclusively via HTTP POST/JSON roundtrips:

```
Browser (UI5 SPA)                          ABAP Backend
       │                                        │
       │──── HTTP GET ─────────────────────────→│  Returns HTML + embedded UI5 app
       │←─── HTML page ────────────────────────│
       │                                        │
       │──── POST {S_FRONT, XX model} ────────→│  1. Parse JSON request
       │                                        │  2. Load draft (session) from DB
       │                                        │  3. Apply model changes (XX → ABAP vars)
       │                                        │  4. Call app->main(client)
       │                                        │  5. App builds view / handles events
       │                                        │  6. Save new draft to DB
       │←─── {S_FRONT, MODEL, VIEW XML} ──────│  7. Return JSON response
       │                                        │
       │  (UI5 renders XML view, binds model)   │
       │──── POST (next event) ───────────────→│  ... repeat
```

**Request JSON** contains `S_FRONT` (event name, draft ID, browser state) and `XX` (two-way binding changes as deltas).
**Response JSON** contains a new draft ID, view XML strings (if view changed), the full JSON model, messages, and follow-up actions.

#### Launchpad Special Case — Request Body Wrapping

The frontend always sends the POST body as `{ "value": <payload> }` (see `app/webapp/core/Server.js`). In standalone mode this envelope arrives intact and `request_parse_body` unwraps it via `slice('value')`.

When the app runs inside the **SAP Fiori Launchpad** (FLP), requests may be routed through the FLP shell or an SAP Gateway proxy. In certain configurations this infrastructure strips the `value` envelope before the request reaches the ABAP ICF handler, so the payload arrives as a plain object without the `value` key.

`request_parse_body` handles both cases defensively:
```abap
DATA(lo_ajson2) = lo_ajson->slice( `value` ).
IF lo_ajson2 IS BOUND.
  lo_ajson = lo_ajson2.   " standalone: unwrap the value envelope
ENDIF.
" launchpad/gateway: no value key → use lo_ajson as-is
```

The Launchpad context is detected afterwards from the parsed request fields:
```abap
result-s_control-check_launchpad = xsdbool(
    result-s_front-search   CS `scenario=LAUNCHPAD`
    OR result-s_front-pathname CS `/ui2/flp`
    OR result-s_front-pathname CS `test/flpSandbox` ).
```

Both scenarios are covered by unit tests in `z2ui5_cl_core_handler.clas.testclasses.abap` (`test_parse_body_with_wrapper` / `test_parse_body_without_wrapper`).

### Layered Design

```
src/
├── 00/   Layer 0: Utilities (AJSON, S-RTTI, framework context/HTTP abstractions)
├── 01/   Layer 1: Core engine (handler, action, binding, model, events, draft service, embedded frontend)
├── 02/   Layer 2: Public API (interfaces, XML view builder, HTTP handler, exit framework)
└── 99/   Obsolete package — retired z2ui5_cl_util* classes (99/01) and built-in popups (99/02), kept for downstream compatibility only
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abap2UI5/abap2UI5](https://github.com/abap2UI5/abap2UI5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
