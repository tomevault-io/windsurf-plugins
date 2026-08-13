---
trigger: always_on
description: This R package implements the Dagster Pipes protocol, enabling R scripts to run as
---


# R package {dagsterpipes}

## Overview

This R package implements the Dagster Pipes protocol, enabling R scripts to run as
external processes orchestrated by Dagster. It communicates bidirectionally with Dagster:
receiving execution context (asset keys, partition info, extras) and reporting back
materializations, asset checks, logs, and custom messages.

## Design Principles

- **Minimal dependencies**: Only {R6} and {jsonlite} are allowed as Imports dependencies.
  All other functionality must use base R only (no {httr}, no {curl}, etc.).
- **Single transport**: File-based message writing only (newline-delimited JSON appended
  to a temp file). This matches the default `PipesSubprocessClient` / `PipesTempFileMessageReader`
  on the Dagster side.
- **Graceful degradation**: When `DAGSTER_PIPES_CONTEXT` is not set (i.e., script is run
  outside Dagster), `open_dagster_pipes()` returns a no-op context that logs to the
  console and silently ignores materializations. This allows scripts to be tested standalone.
- **R6 class system**: Use R6 for stateful objects (PipesContext, message writer channel).

## Dagster Pipes Protocol Specification

### Environment Variables

Dagster injects two environment variables into the subprocess:

| Variable | Purpose |
|----------|---------|
| `DAGSTER_PIPES_CONTEXT` | Base64-encoded, zlib-compressed JSON. Decodes to `{"path": "<tempfile>"}` pointing to context data. |
| `DAGSTER_PIPES_MESSAGES` | Base64-encoded, zlib-compressed JSON. Decodes to `{"path": "<tempfile>"}` pointing to where messages should be written. |

### Decoding Bootstrap Params

```
env_var_value -> base64_decode -> zlib_decompress -> JSON_parse -> {"path": "/tmp/xxx"}
```

In R:
```r
decode_param <- function(encoded) {
  raw_bytes <- jsonlite::base64_dec(encoded)               # {jsonlite}
  decompressed <- memDecompress(raw_bytes, type = "gzip")  # base R
  jsonlite::fromJSON(rawToChar(decompressed), simplifyVector = FALSE)
}
```

Use `jsonlite::base64_dec()` for base64 decoding, `base::memDecompress()` for zlib
decompression, and `jsonlite::fromJSON()` for JSON parsing.

### Context Data Structure

The file at the context path contains JSON:

```json
{
  "asset_keys": ["my_asset"],
  "code_version_by_asset_key": {"my_asset": null},
  "provenance_by_asset_key": {"my_asset": null},
  "partition_key": null,
  "partition_key_range": null,
  "partition_time_window": null,
  "run_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "job_name": "my_job",
  "retry_number": 0,
  "extras": {}
}
```

### Message Format

Every message written to the messages file is a single-line JSON object:

```json
{"__dagster_pipes_version": "0.1", "method": "<method>", "params": {<params>}}
```

### Message Types

#### `opened` — First message, sent on session init
```json
{"__dagster_pipes_version": "0.1", "method": "opened", "params": {}}
```

#### `closed` — Last message, sent on session teardown
```json
{"__dagster_pipes_version": "0.1", "method": "closed", "params": {}}
```

#### `log` — Log message
```json
{
  "__dagster_pipes_version": "0.1",
  "method": "log",
  "params": {"message": "...", "level": "INFO"}
}
```
Valid levels: `"DEBUG"`, `"INFO"`, `"WARNING"`, `"ERROR"`, `"CRITICAL"`

#### `report_asset_materialization` — Report asset completion
```json
{
  "__dagster_pipes_version": "0.1",
  "method": "report_asset_materialization",
  "params": {
    "asset_key": "my_asset",
    "data_version": null,
    "metadata": {
      "row_count": {"raw_value": 1000, "type": "int"},
      "path": {"raw_value": "/data/output.csv", "type": "path"}
    }
  }
}
```

#### `report_asset_check` — Report check result
```json
{
  "__dagster_pipes_version": "0.1",
  "method": "report_asset_check",
  "params": {
    "asset_key": "my_asset",
    "check_name": "row_count_check",
    "passed": true,
    "severity": "ERROR",
    "metadata": {}
  }
}
```

#### `report_custom_message` — Arbitrary payload
```json
{
  "__dagster_pipes_version": "0.1",
  "method": "report_custom_message",
  "params": {"payload": {}}
}
```

### Metadata Value Types

Metadata values are typed:
```json
{"raw_value": <value>, "type": "<type>"}
```

Supported types: `"text"`, `"url"`, `"path"`, `"notebook"`, `"json"`, `"md"`,
`"float"`, `"int"`, `"bool"`, `"dagster_run"`, `"asset"`, `"null"`, `"table"`,
`"table_schema"`, `"table_column_lineage"`, `"timestamp"`, `"__infer__"`

When type is `"__infer__"`, Dagster infers the type from the raw_value.

## Package Architecture

### File Layout

```
R/
  dagsterpipes-package.R     # Package-level docs, @importFrom R6 R6Class, @importFrom jsonlite fromJSON toJSON base64_dec
  utils.R                    # decode_param(), json_serialize() wrapper
  context.R                  # PipesContext R6 class
  message_writer.R           # PipesFileMessageWriterChannel R6 class
  open.R                     # open_dagster_pipes() entry point
  metadata.R                 # pipes_metadata_value() helper, type inference
tests/
  testthat/
    test-utils.R             # Unit tests for base64, JSON, decode_param
    test-context.R           # Tests for PipesContext methods
    test-message_writer.R    # Tests for message writing
    test-open.R              # Integration tests for open_dagster_pipes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joekirincic/dagsterpipes](https://github.com/joekirincic/dagsterpipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
