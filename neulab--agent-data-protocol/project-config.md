---
trigger: always_on
description: This document captures key patterns and best practices for contributing to the Agent Data Protocol repository.
---

# Agent Data Protocol - Repository Guidelines

This document captures key patterns and best practices for contributing to the Agent Data Protocol repository.

## Repository Structure

```
agent-data-protocol/
├── datasets/           # Dataset implementations
│   └── $DATASET_NAME/
│       ├── README.md
│       ├── extract_raw.py
│       ├── raw_to_standardized.py
│       ├── schema_raw.py
│       ├── api.py (required if ApiAction is used)
│       ├── sample_raw.json
│       ├── sample_std.json
│       └── sample_sft/
│           ├── openhands_v0.json
│           └── $AGENT_NAME.json
├── agents/             # Agent-specific SFT converters
├── schema/             # ADP standardized format definitions
├── scripts/            # Utility scripts
└── tests/              # Validation tests
```

## Data Flow Pipeline

```
Raw Dataset      →  Standardized Format  →  Agent Specific SFT Format
     ↓                   ↓                       ↓
sample_raw.json  →  sample_std.json      →  sample_sft/<agent_name>.json
```

## Key Requirements

### Dataset File Naming and Structure
- Every dataset directory must include `README.md`, `extract_raw.py`, `raw_to_standardized.py`, `schema_raw.py`, `sample_raw.json`, `sample_std.json`, and `sample_sft/openhands_v0.json` unless there is a documented reason that the dataset is intentionally incomplete.
- If `sample_std.json` exists, `sample_sft/openhands_v0.json` is required. Additional agent-specific files may live under `sample_sft/` using the exact agent identifier as the filename, such as `sample_sft/sweagent.json`.
- Only these top-level JSON files are allowed in dataset directories:
  - `sample_raw.json`
  - `sample_std.json`
  - `generated_thoughts.json`
- Do not commit `full_raw.json`, `full_std.json`, `full_sft.json`, temporary chunks, downloaded corpora, scratch JSON, or alternate sample files such as `sample_fixed.json`.
- All JSON files MUST be valid JSON and MUST have a trailing newline.

### Generated Samples Must Come From the Pipeline
- Treat `sample_raw.json`, `sample_std.json`, and files under `sample_sft/` as generated artifacts from the dataset scripts, not hand-edited fixtures.
- If a sample fails validation, fix `extract_raw.py`, `raw_to_standardized.py`, `schema_raw.py`, `api.py`, or the relevant agent converter, then regenerate the sample files.
- Do not directly patch sample JSON just to satisfy a failing test unless the same logic is also encoded in the generator that produced it.
- Keep the same records and order across `sample_raw.json`, `sample_std.json`, and each `sample_sft/<agent_name>.json`; the samples should represent the same tasks at each stage, with matching IDs between standardized and SFT files.
- Use small representative samples, normally 3-5 trajectories, that include important edge cases such as tool calls, command output, final answers, and any dataset-specific action types.

### SFT Format Requirements

**Critical**: Messages containing function call patterns MUST use `"from": "function_call"`, not `"from": "gpt"`, `"human"`, or any other role.

Function call patterns that trigger this requirement:
- `<function=`
- `<function_calls>`
- `<invoke name=`

Example correct format:
```json
{
  "from": "function_call",
  "value": "I'll run the command.\n\n<function=execute_bash>\n<parameter=command>ls -la</parameter>\n</function>"
}
```

### Standardized Schema Components

**Actions:**
- `MessageAction`: Text-based assistant communication that is not a tool call or executable code.
- `CodeAction`: Code execution requests such as shell commands or notebook cells.
- `ApiAction`: API/function calls with `function` and `kwargs` fields.

**Observations:**
- `TextObservation`: Text-based responses with `source` set to a schema-allowed source (`user`, `agent`, or `environment`). Do not invent values like `system`, `os`, or `assistant`.
- `WebObservation`: Web page content.

**Versioning:**
- The canonical ADP standardized schema version lives in `schema/version.py` as `SCHEMA_VERSION`.
- `Trajectory` includes a root-level `schema_version`; committed `sample_std.json` files must include the current value explicitly.
- Any schema-impacting Python change under `schema/` must bump `SCHEMA_VERSION`; CI checks this with `scripts/check_schema_version_bump.py`.

### API and Schema Validity
- Every `ApiAction.function` used in `sample_std.json` must be implemented in that dataset's `api.py` with a compatible Python signature.
- Every `ApiAction.kwargs` object must validate by calling the function in `api.py`; include required parameters such as the `message` argument for `finish`.
- If a trajectory has `available_apis`, it must be a top-level list of function names that is a subset of that dataset's `api.py` functions, and every `ApiAction.function` used in that trajectory must appear in the list.
- Only populate `available_apis` for datasets that have `api.py` and whose source data explicitly specifies per-instance tool/API availability. Do not fill it with all functions from `api.py`, and do not infer it merely from the APIs used in the trajectory.
- `schema_raw.py` must faithfully model the raw samples, and `sample_raw.json` must validate against it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neulab/agent-data-protocol](https://github.com/neulab/agent-data-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
