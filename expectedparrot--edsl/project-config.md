---
trigger: always_on
description: This guide is for coding agents and automation using the EDSL command line. Use `ep` in examples. `edsl` and `python -m edsl` are equivalent entry points when `ep` is not installed.
---

# Agent Guide: EDSL CLI

This guide is for coding agents and automation using the EDSL command line. Use `ep` in examples. `edsl` and `python -m edsl` are equivalent entry points when `ep` is not installed.

## Operating Model

The CLI is designed for programmatic use.

- Stdout is a single JSON document.
- Stderr is for diagnostics and logs.
- Commands should not prompt interactively.
- Always parse stdout as JSON and inspect `status`.
- Read `warnings`; lower-level output may be captured there to preserve the JSON envelope.
- Do not scrape prose, progress lines, URLs, or table output from stdout.
- Prefer `.ep` packages for durable EDSL objects.

Success envelope:

```json
{
  "status": "ok",
  "data": {},
  "warnings": []
}
```

Error envelope:

```json
{
  "status": "error",
  "error": {
    "code": "USAGE_ERROR",
    "message": "What failed",
    "suggestion": "What to try next"
  }
}
```

Exit codes:

| Code | Meaning |
| --- | --- |
| `0` | Success |
| `1` | General error |
| `2` | Usage error |
| `3` | Resource not found |
| `4` | Authentication error |
| `5` | Validation error |
| `6` | Remote service error |

## First Commands To Run

Start by discovering the installed CLI rather than guessing flags.

```bash
ep --help
ep info
ep run --help
ep jobs --help
ep results --help
ep schema list
```

`ep info` reports version and configuration diagnostics and redacts credentials.

## Command Inventory

Top-level commands available in the CLI:

| Command | Purpose |
| --- | --- |
| `agents` | Create or transform `AgentList` objects. |
| `auth` | Manage authentication. |
| `balance` | Show Expected Parrot credit balance. |
| `check` | Check Expected Parrot URL and API key connectivity. |
| `clone` | Clone a shared object into a local `.ep` package. |
| `costs` | Track estimated and actual job costs in a ledger. |
| `credits` | Open or return the Expected Parrot credits page. |
| `delete` | Delete a remote object. |
| `humanize` | Create and manage human surveys. |
| `info` | Show version, config, and diagnostics. |
| `inspect` | Inspect local packages/files or remote objects. |
| `jobs` | Manage remote jobs. |
| `metadata` | Read object metadata. |
| `models` | List models or create `ModelList` files. |
| `objects` | Namespace for remote object commands. |
| `open` | Render/open an object as HTML. |
| `profile` | Show authenticated profile. |
| `profiles` | Manage local environment profiles. |
| `pull` | Pull latest remote object data into a package. |
| `push` | Push or patch an object on Expected Parrot. |
| `results` | Query and export `Results`. |
| `run` | Run jobs/questions. |
| `scenarios` | Create or transform `ScenarioList` objects. |
| `schema` | Introspect object and question schemas. |
| `search` | Search shared Expected Parrot objects. |
| `settings` | Show remote EDSL settings and rate-limit diagnostics. |
| `share` | Share an object with a user. |
| `shared` | List users an object is shared with. |
| `surveys` | Create and edit `Survey` objects. |
| `unpack` | Unpack a `.ep` package for inspection. |
| `unshare` | Remove user access to an object. |
| `unzip` | Alias for `ep unpack`. |
| `update-metadata` | Update remote metadata without object changes. |
| `validate` | Validate questions, surveys, and jobs. |

Many remote object commands are available both at top level and under `ep objects`, for example `ep search` and `ep objects search`.

## Schemas And Validation

Use schemas before constructing JSON by hand.

```bash
ep schema show --class Survey
ep schema show --class AgentList
ep schema show --class ScenarioList
ep schema show --class ModelList
ep schema show --class Jobs
ep schema show --class Results
ep schema show --question_type free_text
ep schema show --question_type multiple_choice
ep schema show --question_type linear_scale
```

Validate objects before running them.

```bash
ep validate --file survey.json
ep validate --file jobs.json
ep validate --json '{"type":"free_text","question_text":"Say hi"}'
```

If validation fails, use the JSON error envelope. Do not continue by trying to run the same invalid input.

## Object Formats

The CLI accepts two broad formats:

- JSON serialized objects, usually from `.to_dict()`.
- Git-backed `.ep` packages.

Use `.ep` packages when you need stable files that can be inspected, pushed, pulled, versioned, or reopened. Use JSON for lightweight generated input or one-off tests.

Common object types:

- `Survey`: questions and survey flow.
- `AgentList`: agent traits and instructions.
- `ScenarioList`: scenario data.
- `ModelList`: model selections.
- `Jobs`: complete run specification combining survey, agents, scenarios, and models.
- `Results`: completed run output.

Inspect objects:

```bash
ep inspect survey.ep
ep inspect jobs.ep
ep metadata results.ep
```

## Building Inputs

Prefer creating complete `Jobs` objects when possible. They contain the full execution specification.

Common patterns:

```bash
# Run a single ad hoc question.
ep run --question "What is 2+2?" --model gpt-4o --output results.ep

# Run a saved Jobs package.
ep run jobs.ep --output results.ep

# Run a saved Jobs JSON file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [expectedparrot/edsl](https://github.com/expectedparrot/edsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
