---
trigger: always_on
description: [English](./AGENTS.md) | [简体中文](./AGENTS.zh-CN.md)
---

# AGENTS.md

[English](./AGENTS.md) | [简体中文](./AGENTS.zh-CN.md)

This file guides AI/Agents on how to use the rust_sap_rfc service.

## What it is

rust_sap_rfc is a **SAP NWRFC → REST gateway**: it exposes SAP RFC/BAPI function modules as HTTP endpoints. Through it, you (the AI) can search, inspect, and invoke function modules in an SAP system without installing an SAP client.

- **Project**: https://github.com/Jack-Liang/sap-for-agents
- **Issues**: https://github.com/Jack-Liang/sap-for-agents/issues

The service listens on `http://127.0.0.1:3000` by default (override with `SAP_LISTEN_ADDR`).

## Authentication (optional)

By default the service is **unauthenticated** (local access). If the deployer sets the `SAP_API_KEY` environment variable, all `/api/*` endpoints require the `Authorization: Bearer <token>` request header:

```bash
curl -H "Authorization: Bearer <SAP_API_KEY>" http://127.0.0.1:3000/api/functions/BAPI_USER_GETLIST
```

- Missing or wrong token → `401 {"code":401,"message":"..."}`.
- The probes `/health`, `/ready` and the public pages `/`, `/agents.md` are **always unauthenticated** (no token required).
- Whether auth is enabled is decided by the deployer. The default local environment is usually unauthenticated — try without a token first, and request one from the deployer if you receive a 401.

## What you can do

| Goal | Endpoint |
|------|-----------|
| Don't know which functions exist → fuzzy search by name | `POST /api/functions/search` |
| Know the function name, want to know how to fill parameters | `GET /api/functions/{name}` |
| Want full function documentation (purpose, examples) | `GET /api/functions/{name}/doc` |
| Want the fields of a table/structure | `GET /api/ddic/type/{name}` |
| Want to understand a field's meaning and valid values | `GET /api/ddic/field/{table}/{field}` |
| Want the function's ABAP source (how it's implemented) | `GET /api/functions/{name}/source` |
| Want the source of a program/report/include | `GET /api/programs/{name}/source` |
| Want to read transparent table data (without calling RFC_READ_TABLE directly) | `POST /api/table/read` |
| Want ABAP short-dump info: list, full ST22 text (What happened/Error analysis) | `GET /api/adt/runtime/dumps` |
| Want to read/write ABAP class sources and other ADT (Eclipse tooling) resources | `ANY /api/adt/{path}` |
| **Actually invoke an SAP function** | `POST /api/rfc` |

## Standard workflow

Most tasks follow five steps: **search → inspect interface → read docs → view source → invoke**:

```
1. Search functions  POST /api/functions/search      Find the target function name
2. Inspect interface GET  /api/functions/{name}      See parameter names, types, directions
3. Read docs         GET  /api/functions/{name}/doc  Understand purpose, constraints, examples
4. View source       GET  /api/functions/{name}/source  Understand the implementation (optional)
5. Invoke            POST /api/rfc                   Fill parameters per the interface and execute
```

> Do not skip step 2 and invoke directly — SAP parameter names are case-sensitive and must be uppercase, and the type (CHAR/INT/BCD...) determines how to pass values. Inspecting the interface first avoids 90% of parameter mistakes.

## Endpoint quick reference (copyable examples)

### 1. Search functions

```bash
curl -X POST http://127.0.0.1:3000/api/functions/search \
  -H "Content-Type: application/json" \
  -d '{"pattern":"BAPI_USER_*","max_results":10}'
```

- `pattern`: function name wildcard; `*` matches anything. E.g. `BAPI_*`, `RFC_*`.
- Returns a `functions` array; each item has `name` / `group` / `description`.
- No match returns `200 {"count":0,"functions":[]}` (**not an error**).

### 2. Inspect function interface

```bash
curl http://127.0.0.1:3000/api/functions/BAPI_USER_GETLIST
```

Returns **all parameters** of the function; each parameter has:
- `name`: parameter name (**always use this exact uppercase name when passing it**)
- `type`: `CHAR` / `INT` / `STRUCTURE` / `TABLE` / `BCD` / `DATE` ...
- `direction`: `IMPORT` (you fill) / `EXPORT` (return value) / `TABLES` (in or out)
- `length`: character length (for CHAR/NUM/DATE etc.)
- `optional`: whether it can be omitted
- `description`: parameter description
- `fields`: for STRUCTURE/TABLE, lists the nested fields

> Namespaced function names (containing `/`, e.g. `/SDF/EWA_GET_ABAP_DUMPS`) are supported.
> In URL paths, use either the raw form (`/api/functions//SDF/EWA_GET_ABAP_DUMPS`) or the
> percent-encoded form (`/api/functions/%2FSDF%2FEWA_GET_ABAP_DUMPS`); in JSON bodies
> (e.g. `func_name` of `/api/rfc`), pass the name as-is.

### 3. Read function documentation

```bash
curl 'http://127.0.0.1:3000/api/functions/BAPI_USER_GETLIST/doc?lang=EN'
```

Returns `short_text` (short description), `long_text` (full SE37 documentation, may be long), and `parameter_docs` (per-parameter descriptions). If `lang` is omitted, the `SAP_LANG` environment variable is used (default EN).

> Not all functions have long documentation. An empty `long_text` is normal — read `parameter_docs` instead.

### 4. Inspect DDIC table/structure fields

```bash
curl http://127.0.0.1:3000/api/ddic/type/BAPIRET2
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jack-Liang/sap-for-agents](https://github.com/Jack-Liang/sap-for-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
