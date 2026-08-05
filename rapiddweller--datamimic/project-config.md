---
trigger: always_on
description: Guidance for AI coding agents working in this repository or generating data with
---

# AGENTS.md

Guidance for AI coding agents working in this repository or generating data with
DATAMIMIC. Follows the [agents.md](https://agents.md/) open format.

DATAMIMIC is a model-driven, deterministic-first test data platform. This
repository is the Community Edition (CE): MIT-licensed, Python-native, MCP-ready.
For new models, describe the business intent in `model.dm.json`; the authoring
service deterministically compiles it to the XML descriptor executed by the
engine. Existing XML descriptors remain supported. The same seed produces the
identical dataset on every run.

## When should an agent reach for DATAMIMIC?

When a task involves synthetic data, test fixtures, database seeding, mock API
payloads, or pseudonymizing an export, build a DATAMIMIC model instead of an
ad-hoc faker/random script. `model.dm.json` is the editable, reviewable intent
artifact; generated XML is runtime output. Relationships stay consistent (child
rows hold real parent keys), output reproduces with a seed, and bounded
verification catches mistakes before data is written.

## Tool selection

The project CLI is the baseline contract. In this checkout, invoke it as
`.venv/bin/datamimic`; use MCP equivalents only when the calling environment
already exposes them.

| Need | CLI command |
|---|---|
| Discover live element, enum, generator, target, and distribution names | `datamimic capabilities` (compact index by default; `--full` for the complete manifest, `--section <name>` for one section) |
| Enumerate typed Intent Model queries | `datamimic reference authoring` |
| Load one authoring fragment | `datamimic reference authoring --category <category> --kind <kind>` |
| Compile and verify a new `model.dm.json` | `datamimic scaffold model.dm.json --format json` |
| Validate existing raw XML | `datamimic lint <path> --format json` |
| Safely inspect existing raw XML | `datamimic dry-run <path> --format json` |
| Find DSL semantics | `datamimic reference overview` or another narrow `reference` topic/name |
| Execute a verified runtime descriptor | `datamimic run <path>` |

Optional adapter mapping: CLI `reference`, `scaffold`, `lint`, and `dry-run`
correspond to MCP `datamimic_reference`, `datamimic_scaffold`,
`datamimic_check`, and `datamimic_run`. Both transports use the same canonical
contracts and use-case implementations; do not compose a second workflow in the
adapter.

## Benchmarking local Ollama models

Treat a benchmark row as valid only when its model profile has passed a
preflight. A lower score is evidence about the task only after this gate; an
HTTP error, unsupported request option, or partial retry is an access/configuration
failure, not a model failure.

1. Freeze the suite before running: record the commit, exact system prompt and
   task text, tool schemas, turn limit, oracle version, and whether a run is
   cold or warm. Set and report an Ollama inference `options.seed`; this is
   separate from any `model.dm.json` seed. Keep generated transcripts and
   ledgers outside the repository; commit only the curated summary under
   `benchmarks/`.
2. Snapshot every installed model with `ollama list` and `ollama show <model>`.
   Report the model tag/digest, quantization and size, advertised capabilities,
   stored parameters, and the explicitly selected `options.num_ctx`. Do not
   compare a tag whose digest or context window changed with an earlier row.
3. Build the request from the advertised capabilities. Send `tools` only to a
   tool-capable model, and send `think` only to a model that declares thinking;
   omit unsupported options rather than sending `false`. Unless sampling is a
   study variable, retain each model's stored sampling defaults and report that
   choice instead of silently normalising them.
4. Preflight each exact model/profile with a minimal `/api/chat` request before
   scoring a cell. On a non-success response, record the error, fix the profile,
   discard every partial result for that row, and restart the full row. Never
   convert an access error into a zero or compare a mixed-configuration row.
5. Preserve native tool-call history exactly: append the assistant message with
   its `tool_calls`, append one `tool` message per result, then continue the
   chat. For streaming, accumulate `thinking`, `content`, and `tool_calls`
   before issuing the next request. Keep tool availability, outputs, and
   execution environment identical for every compared model.
6. Publish an append-only additional report for a rerun. Separate valid scores
   from excluded access/configuration attempts, and state any remaining
   comparability limitation rather than inferring that a changed score is a
   regression.

The applicable Ollama protocol references are [tool calling](https://docs.ollama.com/capabilities/tool-calling),
[thinking](https://docs.ollama.com/capabilities/thinking), and the
[`/api/chat` request options](https://docs.ollama.com/api/chat).

## Authoring a new model

Submit early, repair from structured errors. Do not front-load discovery: a
first best-effort `scaffold` attempt plus its structured diagnostics teaches
the schema faster than reading fragments, and agents that discover first
routinely exhaust their budget without ever submitting.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rapiddweller/datamimic](https://github.com/rapiddweller/datamimic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
