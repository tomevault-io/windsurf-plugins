---
trigger: always_on
description: How to drive coreai-onnx programmatically. Every command accepts `--json` and
---

# coreai-onnx — Agent Guide

How to drive coreai-onnx programmatically. Every command accepts `--json` and
prints exactly one JSON envelope on stdout; exit codes and error codes are
stable contracts (see Stability, below).

## Install

| Goal | Command | Platforms |
|---|---|---|
| Convert only | `pip install coreai-onnx` | any |
| + validation & precision checks | `pip install "coreai-onnx[verify]"` | any (the precision check itself needs macOS 27+) |

Executing a converted `.aimodel` requires macOS 27+ / iOS 27+ (Core AI
framework). Conversion itself runs anywhere.

## Discover capabilities

    coreai-onnx schema --json

Returns the full machine-readable contract: commands, flags, error codes,
warning codes, exit codes, the supported-op list, and runtime requirements.
Treat this as the source of truth — it is generated from the same tables the
CLI emits from, so it cannot drift from behavior.

## MCP server

Prefer native tool calls? `pip install "coreai-onnx[mcp]"` and register the
stdio server `coreai-onnx-mcp` with your MCP client:

```json
{"mcpServers": {"coreai-onnx": {"command": "coreai-onnx-mcp"}}}
```

It exposes `inspect_model`, `convert_model`, `verify_model`, and
`get_schema`, each returning the same envelope documented here — branch on
`status`/`error.code` exactly as below. Exit codes do not exist over MCP.
Boolean parameters replace the CLI's negative flags (`optimize=false` ≙
`--no-optimize`); `entrypoint` ≙ `--name`.

## The envelope

Every `--json` invocation prints exactly one object of this shape on stdout:

```json
{
  "schema_version": 1,
  "command": "<convert|inspect|verify|schema>",
  "status": "<ok|error>",
  "result": { "...": "command-specific payload, or null" },
  "warnings": [ { "code": "...", "message": "..." } ],
  "error": { "code": "...", "message": "...", "details": {}, "hint": "..." }
}
```

Rules:

- `status` is `"error"` if and only if `error` is non-null.
- `result` may be partial or null on error — read what is present, do not
  require every key.
- Non-finite precision metrics serialize as strings — `"inf"`, `"-inf"`, or
  `"nan"` (JSON has no literals for them).
- `warnings` is always an array (possibly empty); each entry has a stable
  `code` and a human-readable `message`.

## Canonical workflow

The examples below are real, unedited CLI output (a single-`Relu` model and a
single-`Det` model, converted in a temp directory on macOS with onnxruntime
installed).

1. **Check coverage first** — `coreai-onnx inspect model.onnx --json`

   ```json
   {
     "schema_version": 1,
     "command": "inspect",
     "status": "ok",
     "result": {
       "model_path": "/var/folders/f9/k__xp4rn4_97h3q7p8597k840000gn/T/tmp6ccnaupb/relu.onnx",
       "total_nodes": 1,
       "convertible": true,
       "ops": [
         {
           "op": "Relu",
           "count": 1,
           "supported": true
         }
       ],
       "unsupported": []
     },
     "warnings": [],
     "error": null
   }
   ```

   Gate on `result.convertible`. Exit code 1 with `status: "ok"` means
   "analyzed fine, not convertible" — read `result.unsupported`.

2. **Convert** — `coreai-onnx convert model.onnx -o model.aimodel --json`

   ```json
   {
     "schema_version": 1,
     "command": "convert",
     "status": "ok",
     "result": {
       "output_path": "/var/folders/f9/k__xp4rn4_97h3q7p8597k840000gn/T/tmp6ccnaupb/relu.aimodel",
       "total_nodes": 1,
       "optimized": true,
       "validated": true,
       "repairs": [],
       "precision": {
         "passed": true,
         "rtol": null,
         "atol": null,
         "min_psnr": null,
         "compute_unit": null,
         "seed": 0,
         "outputs": [
           {
             "name": "out0",
             "max_abs_error": 0.0,
             "max_rel_error": 0.0,
             "psnr": "inf",
             "passed": true,
             "expected_nonfinite": 0
           }
         ]
       }
     },
     "warnings": [],
     "error": null
   }
   ```

   On success, `result.precision` carries the ONNX Runtime comparison when it
   ran; `warnings` explains skipped steps (`onnxruntime_missing`,
   `platform_no_runtime`).

   Add `--repair` to auto-fix documented Core AI runtime limitations (e.g.
   float16) with known-safe, parity-verified rewrites; applied fixes are listed
   in `result.repairs`.

3. **On failure, branch on `error.code`** — here a model containing the
   unsupported `Det` op (exit code 1):

   ```json
   {
     "schema_version": 1,
     "command": "convert",
     "status": "error",
     "result": null,
     "warnings": [],
     "error": {
       "code": "unsupported_ops",
       "message": "The following ONNX ops have no Core AI lowering:\n  Det (1 node(s), e.g. node_0)\n\nRegister a custom lowering to proceed:\n    @converter.register_onnx_lowering(\"Det\")\n    def lower(values_map, node, loc): ...\nRun `coreai-onnx inspect <model>` for a full coverage report.",
       "details": {
         "missing": {
           "Det": [
             "node_0"
           ]
         }
       },
       "hint": "Register a custom lowering with @converter.register_onnx_lowering, or run `coreai-onnx inspect <model>` for a full coverage report."
     }
   }
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devin-lai/coreai-onnx](https://github.com/devin-lai/coreai-onnx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
