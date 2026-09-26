---
trigger: always_on
description: <!-- SPDX-License-Identifier: Apache-2.0 -->
---

<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- SPDX-FileCopyrightText: 2026 SubLang International <https://sublang.ai> -->

# gemini: Gemini CLI Adapter

## Intent

This package lets a consumer of the agent-adapter contract run Gemini CLI as a spawned child process whose NDJSON stream becomes unified events, per [DR-002](../../decisions/002-unified-event-stream-and-adapter-interface.md).
It owns how a portable request becomes a Gemini CLI invocation and how that invocation's stream, exit code, policy files, and telemetry become unified events, permission rules, resume continuity, and token accounting, not what a caller does with them and not the CLI's own behavior.
Its requirements are stated in this project's `AgentAdapter`, `AgentEvent`, `AgentOptions`, `PermissionPolicy`, `DonePayload`, and `Cligent` vocabulary, which the engine defines and without which this adapter's behavior cannot be stated.

## External Behavior

### Adapter Identity

### gemini-1

The adapter shall implement `AgentAdapter` with `agent: 'gemini'`.

### gemini-18

The adapter module shall load without a Gemini SDK package, because it communicates with the `gemini` executable.

### Availability

### gemini-2

When `isAvailable()` uses the adapter's default probe, it shall classify the `gemini` executable through this matrix:

| Probe outcome | Result |
| --- | --- |
| `gemini --version` succeeds within 5,000 ms and [[engine-25](../engine.md#engine-25)] does not find the reported version below the supported floor | `true` |
| the executable is missing, exits nonzero, times out, or reports a version below that floor | `false` |

### Process Lifecycle

### gemini-3

When `run()` starts Gemini CLI, the adapter shall execute one headless stream invocation according to this flow:

1. Spawn `gemini` with `--output-format`, `stream-json`, and the option arguments selected by [[gemini-7](#gemini-7)].
2. Put the arbitrary prompt in the final joined token `--prompt=<prompt>`, so Gemini CLI 0.50 does not reinterpret a leading-dash prompt as an option [[4]].
3. Pipe stdout through `parseNDJSON()` per [[ndjson-1](../ndjson.md#ndjson-1)].

### gemini-19

While no native `result` has selected a terminal and the run is not aborted, when settings or policy setup rejects, spawn throws synchronously, the child exposes no stdout, the child reports an asynchronous launch error, or stdout iteration throws, the adapter shall emit `init` first if needed, then a non-recoverable `error` with code `GEMINI_STREAM_ERROR` and the thrown `Error` message or `Gemini adapter failed while reading stream`, followed by terminal `done` with `status: 'error'`, the normal-terminal resume selection in [[gemini-9](#gemini-9)], elapsed duration, usage containing only [[gemini-27](#gemini-27)]'s tool count, and no result.

### gemini-5

While no native `result` has selected a terminal, when the child closes normally, the adapter shall select its terminal outcome from this matrix, using elapsed duration, the resume token in [[gemini-9](#gemini-9)], and usage containing only [[gemini-27](#gemini-27)]'s tool count in every row:

| Close state | Terminal outcome |
| --- | --- |
| the run requested abort, or signal is `SIGTERM` | `done.status: 'interrupted'`; omit the result even when trimmed stderr is non-empty [[gemini-8](#gemini-8)] |
| code `0` | `done.status: 'success'`; trimmed stderr, when non-empty, is the result |
| code `53` | `done.status: 'max_turns'`; trimmed stderr, when non-empty, is the result |
| code `1`, code `42`, another nonzero or null code, or any other signal | a non-recoverable `GEMINI_EXIT_ERROR` followed by `done.status: 'error'`, both carrying trimmed stderr or `Gemini CLI exited with code <code-or-null> without a result event` |

### Environment

### gemini-10

When the adapter builds the child environment, it shall select `GEMINI_CLI_TRUST_WORKSPACE` from this matrix:

| Parent environment | Child value |
| --- | --- |
| variable absent | `'true'` |
| variable present, including `'false'` or an empty string | the existing value unchanged |

### Event Normalization

### gemini-4

When the adapter receives a parsed Gemini stream object before a terminal has been selected, it shall dispatch it according to this matrix:

| Non-empty `type` | Outcome |
| --- | --- |
| `init` | the handshake selected by [[gemini-20](#gemini-20)] |
| `message` | the text selection in [[gemini-21](#gemini-21)] |
| `tool_use` or `tool_call_request` | one `tool_use` selected by [[gemini-22](#gemini-22)] |
| `tool_result` or `tool_call_response` | one `tool_result` selected by [[gemini-23](#gemini-23)] |
| `error` | one `error` selected by [[gemini-24](#gemini-24)] |
| `result` | the terminal selection in [[gemini-25](#gemini-25)] |
| absent, empty, non-string, or any other value | no event beyond any first-event `init` that [[gemini-20](#gemini-20)] requires for a non-empty unknown type |

### gemini-20

When a run selects its `init`, the adapter shall emit exactly one handshake according to this matrix, with tools and capabilities selected by [[gemini-16](#gemini-16)]:

| Stream state | Emission and payload |
| --- | --- |
| first parsed object is native `init` | emit from that object |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sublang-ai/cligent](https://github.com/sublang-ai/cligent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
