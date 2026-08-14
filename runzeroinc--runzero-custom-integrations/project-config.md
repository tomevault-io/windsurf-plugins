---
trigger: always_on
description: This document provides guidance for creating custom integration scripts for runZero.
---

# Custom Integration Agents

This document provides guidance for creating custom integration scripts for runZero.

## Goal
Create a custom integration script to import assets into runZero from a third-party service (Inbound) or export runZero assets to a third-party service (Outbound).

## Directory Structure
Each integration must be placed in its own directory at the root of the repository.

```
repo-root/
├── <integration-name>/
│   ├── <integration-name>.star  # The main script (also carries the integration metadata)
│   └── README.md                # Documentation
```

### 1. Integration metadata (embedded `CONFIG` block)

Every new script must declare `CONFIG = {...}` as its first top-level statement
of the file. The platform extracts this block (via a strict literal-only
Starlark walk) to render the credential form, validate user input, apply
defaults, and route secret fields through encrypted storage. Only literal
expressions are permitted on the right-hand side — no function calls, no
variable references, no arithmetic. The only exception is
`CONFIG["includes"]`, which may reference allowlisted shared option-set
identifiers such as `OPTIONS_TLS` and `OPTIONS_HTTP`.

**Format:**
```python
CONFIG = {
    "id": "runzero-example",
    "name": "Integration Name",
    "type": "inbound",
    "description": "Short summary shown in the catalog.",
    "version": "26052700",
    "minVersion": "5.0.260723.0",
    "params": [
        {
            "key": "client_id",
            "label": "Client ID",
            "type": "string",
            "required": True,
        },
        {
            "key": "client_secret",
            "label": "Client secret",
            "type": "secret",
            "required": True,
        },
    ],
    "includes": {
        "tls_": OPTIONS_TLS,
        "http_": OPTIONS_HTTP,
    },
}

load("runzero.types", "ImportAsset")
# ...rest of script
```

**Required rules:**
- `CONFIG` must be the first top-level statement. Comments and blank lines may precede it; `load(...)`, constants, and other statements may not.
- All values must be literals (`True`/`False`/`None`, strings, ints, floats, lists, tuples, dicts with string keys, negated numbers via unary `-`).
- `id` must be a stable lower-case integration identifier, e.g. `runzero-tailscale`.
- `version` must use the integration version string for the target release, e.g. `26052700`.
- Scripts in this v2 library must set `minVersion` to `5.1.0` or later. Before the script runs, the Explorer version is compared against it and older releases fail with a clear upgrade message. Development builds using the `0.0.0` sentinel skip the check.
- `type` must be `inbound`, `outbound`, or `internal`.
- Each `params[].key` must match `^[a-zA-Z_][a-zA-Z0-9_]*$` and must match the kwarg name the script reads.
- `type: "secret"` (or `secret: True`) marks the field for masked input and log redaction; never log or print these values, and never set a `default` on them. All dynamic credential fields are encrypted at rest.
- `includes` expands shared option sets with the dict key as a prefix, for example `{"src_tls_": OPTIONS_TLS, "dst_http_": OPTIONS_HTTP}`.

**Supported top-level CONFIG fields:** `id`, `name`, `type`, `description`, `version`, `minVersion`, `params`, `includes`, `rejectUnknown`, `atLeastOneOf`, `exactlyOneOf`, `validationMode`.

**Supported param types:** `string`, `secret`, `int`, `float`, `bool`, `enum` (requires `options`), `url`, `textarea`, `json`.

**Supported param fields:** `key`, `label`, `description`, `type`, `required`, `secret`, `default`, `placeholder`, `options`, `multi`, `min`, `max`, `pattern`, `caseInsensitive`, `aliases`, `dependsOn`, `visibleIf`, `visibleIfValue`, `requiredIf`, `requiredIfValue`, `group`.

Enum aliases are normalized to their canonical `options` value before `main`
runs. CONFIG-based integrations reject unknown kwargs. Use
`"validationMode": "compile"` only for templates and integrations that use
direct protocols such as SSH, SMB, WMI, WinRM, or SQL; HTTP integrations should
omit it and use the default HTTP wiring validation.

Direct-protocol modules execute on the selected Explorer and intentionally can
connect to internal addresses available from that Explorer. Treat the script
and its credentials as privileged discovery configuration, and scope the
Explorer and credential to the intended source system.

### 2. `<integration-name>.star`
This is the main script written in Starlark. Name it after the
integration directory (e.g. `tailscale/tailscale.star`).

## Script Development

The CONFIG model and standard helper modules are designed to be easy to use
with external authoring tools, including LLM-based editors. runZero does not run
an LLM in the Console, Explorer, or Starlark sandbox; generated scripts are
reviewed and executed like any other source file.

### Language
The script is written in **Starlark**, a Python-like language with some key differences:
*   **No Exceptions**: Use return values and status codes for error handling.
*   **No f-strings**: Use `"{}".format(var)` for string interpolation.
*   **Limited Standard Library**: Only specific built-ins and loaded libraries are available.

### Entrypoint
The script must define a `main` function.

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runZeroInc/runzero-custom-integrations](https://github.com/runZeroInc/runzero-custom-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
