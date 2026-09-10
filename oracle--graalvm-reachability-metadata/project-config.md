---
trigger: always_on
description: Forge treats an agent as a replaceable editing backend behind a small Python
---

# AR-agent-api: Forge agent API and backend adapters

Forge treats an agent as a replaceable editing backend behind a small Python
interface, preserving the boundary in §AR-forge-strategy-agent-boundary.
§FS-forge-agent-runtime-selection requires every adapter to expose the same
editing boundary.
Workflow engines decide what work to do next; agents only send prompts,
maintain or clear conversation context, report token usage, and run
agent-visible test commands, with bundles wiring the chosen backend into a
strategy (§FS-forge-predefined-strategy-contract).

## 1. Agent API

[`ai_workflows/agents/agent.py`](../../ai_workflows/agents/agent.py) defines the
`Agent` base class and registry. Concrete backends register with
`@Agent.register("<name>")`, and strategy loading resolves the configured
backend by the strategy's `agent` field (see
§FS-forge-predefined-strategy-contract).

The interface is deliberately narrow:

| Method or property | Contract |
| --- | --- |
| `send_prompt(prompt)` | Send one prompt to the current agent session and return the agent's text response. |
| `fork(prompt)` | Branch from the current conversation and send the prompt in the child session. |
| `compact_fork(prompt)` | Branch with reduced context when the backend supports compaction. |
| `clear_context()` | Drop agent-side conversation state before an independent step or run. |
| `run_test_command(test_cmd)` | Execute a deterministic test command and return diagnostics in the form expected by the agent. |
| `graphify(source_dirs)` | Build optional graph context from read-only source directories. |
| `total_tokens_sent` / `total_tokens_received` / `cached_input_tokens_used` | Expose token accounting for run metrics and cost reporting. |

This API keeps workflow behavior out of backend adapters. A new backend should
implement the registry key, the prompt/session methods, token counters, and
test-command bridge, then
be selected through strategy data (§AR-forge-workflow-strategy-config) instead
of changing workflow drivers or workflow engines.

## 2. Backend implementations

The registry exposes `claude-code`, `pi`, `codex`, and `opencode`. Each adapter
starts its CLI unattended, with the tools that backend provides, so a repair
step can reproduce the failure it was given rather than editing blind.

`source_context.url_fetch_agent_command` builds the command string for URL-field
discovery, which Gradle invokes directly instead of through an adapter.

[`PiAgent`](../../ai_workflows/agents/pi_agent.py) registers the `pi` backend and
drives Pi through [`PiRpcClient`](../../ai_workflows/agents/pi_rpc_client.py), a
thin subprocess wrapper around `pi --mode rpc`. The client starts Pi in the
workflow working directory, applies optional `--provider`, `--model`,
`--session-dir`, and persistent system-prompt flags, sends prompt JSON over
stdin, and reads structured RPC events from stdout until the agent reports the
turn is complete.

Each successful Pi turn returns a `PromptResult` with:

- the assembled assistant text from `message_update` deltas;
- the active Pi session file from `get_state`;
- token totals from `get_session_stats`;
- the RPC transcript used for durable debugging.

`PiAgent.send_prompt` continues the current session when one exists, updates the
session file after every turn, computes per-turn input, output, and cache-read
token deltas, writes a coordinate-scoped session log, and returns only the text
response to the workflow engine. Durable prompt, response, session, and failure
logs are part of the Forge diagnostic contract defined in
§FS-durable-generation-logs.

`PiAgent.fork` uses Pi's `--fork <session>` support to create a child
conversation while preserving the parent counters as the baseline for the child.
`compact_fork` currently delegates to `fork` because Pi RPC does not expose a
documented compaction-aware fork operation. `clear_context` drops the stored
session path and token baselines so the next prompt starts as an independent Pi
session.

`PiAgent.run_test_command` delegates Gradle execution to the
[shared test runner](../../utility_scripts/gradle_test_runner.py) instead of
asking Pi to choose shell behavior. That preserves the architecture: the
workflow engine chooses the gate (§AR-forge-workflow-engine), deterministic
utilities run it, and the agent receives diagnostics for the next edit cycle,
keeping the strategy/agent boundary intact (§AR-forge-strategy-agent-boundary).

## 3. Runtime roles and defaults

The three roles are configured from different places. `FORGE_ANALYSIS_AGENT` /
`FORGE_ANALYSIS_FAMILY` / `FORGE_ANALYSIS_MODEL` / `FORGE_ANALYSIS_PROVIDER`
select recovery, style, native-test, post-generation, and review work; the
defaults are Codex with `gpt-5.6-luna` and high reasoning, and Claude Code
answers to its `sonnet` model alias.
`FORGE_SETUP_*` selects artifact-URL discovery and the library-preparation
preflight. The roles do not read each other: what a role leaves unset comes from
the shared defaults, so retuning one never moves the other. The do-work loop exports
these values unchanged across its self-update boundary (§AR-do-work-loop).

Every backend takes a reasoning effort, in its own spelling: Codex

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oracle/graalvm-reachability-metadata](https://github.com/oracle/graalvm-reachability-metadata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
