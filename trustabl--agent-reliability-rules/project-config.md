---
trigger: always_on
description: These instructions apply to any work on the detection rule packs in this
---

# Instructions for Claude — Trustabl detection rules

These instructions apply to any work on the detection rule packs in this
repository. The packs here are pulled by the
[Trustabl engine](https://github.com/trustabl/agent-reliability-analyzer) at
scan time; the rule **schema** and **predicate** implementations, plus the
per-rule test harness, live in that engine repository. Links below labeled
"engine repo" point there.

## Engine/rules split — read this first

- **Rules** (the `.yaml` packs) live **here**.
- **Schema, predicates, evaluator, loader, and the per-rule fire/silent test
  table** live in the **engine repo** under `internal/rules/`.
- The engine validates rules against its own copy of these packs at
  `testdata/rules-fixture/` (the test fixture). Until the test harness is
  pointed directly at this repository, **a rule change here must be mirrored
  into the engine's `testdata/rules-fixture/` with its fire/silent test cases**,
  or the engine's `TestPolicyRules_AllRulesCovered` guard will not cover it.
- There is no `go test` to run *in this repo* — it has no Go. Validation runs
  in the engine repo (`go test ./...`).

## Required reading order before editing

1. Schema reference (engine repo): [`internal/rules/schema.yaml`](https://github.com/trustabl/agent-reliability-analyzer/blob/main/internal/rules/schema.yaml) — authoritative field reference.
2. [`README.md`](README.md) — conventions in this repo.
3. The closest existing rule to what's being asked for — pattern example.

Do not skip step 1.

## Hard rules

- **Never invent YAML keys.** The schema is closed (`KnownFields(true)`). If a
  field you want does not exist in the engine's
  [`internal/rules/schema.go`](https://github.com/trustabl/agent-reliability-analyzer/blob/main/internal/rules/schema.go),
  extending the schema is a four-file change **in the engine repo**
  (schema.go + predicates.go + evaluator.go + schema.yaml), gated by a
  `manifest.yaml` `schema_version` bump here. Make the engine changes in one
  commit.
- **Never change a rule's `id` after it has shipped.** IDs are external
  identifiers; downstream consumers cite them, and the engine folds the
  resolved pack into the scan `ScanID`.
- **Never duplicate a rule ID across files.** The loader rejects this at
  startup.
- **Never widen `applies_to` across SDKs casually.** A rule's `explanation` /
  `fix` text is usually SDK-specific. Adding `openai_tool` to a Claude-SDK rule
  (or vice versa) makes the user-facing text lie. If a cross-SDK pattern is
  genuinely needed, author a separate rule under that SDK's category
  (`<sdk>_sdk/<topic>.yaml`) with framing that matches the target SDK.
- **Never write rules at `info` severity.** Reserved.

## Required fields per rule

Every rule MUST set: `id`, `title`, `severity`, `confidence`, `applies_to`,
`match`, `explanation`, `fix`. The loader refuses to start the scanner if any
are missing — this surfaces as a `scan: ...` error in the CLI.

`language:` is OPTIONAL but state it explicitly — defaults to `python` when
omitted. For TypeScript / JavaScript / Go rules, set it explicitly:
`language: typescript`. Note: only Python tool discovery is plumbed into the
engine today, so non-python rules will load and validate but never fire until
the matching parser ships.

## Per-scope `applies_to` values

The `applies_to` list constrains which discovered entities a rule fires
against. Pick values from the table for the scope you're targeting.

**`scope: tool`** — receives a `ToolDef`; `applies_to` is matched against
`ToolDef.Kind`:

| `applies_to` value   | Matches                                       |
| -------------------- | --------------------------------------------- |
| `openai_tool`        | `@function_tool`-decorated Python function    |
| `claude_sdk_tool`    | `@tool` / `@claude_tool` / `claude_agent_sdk` |
| `mcp_tool`           | `@server.tool`, `@mcp.tool`, `.register_tool` |
| `shell_invocation`   | Bare function that calls `subprocess.*` etc. (no rules currently target this — OSH-* moved to a closed-source project) |
| `langchain_tool`     | `@tool` / `StructuredTool` / `tool(...)` (LangChain, LangGraph) |
| `crewai_tool`        | `@tool` / `BaseTool` subclass (CrewAI)        |
| `pydantic_ai_tool`   | `@agent.tool` / `@agent.tool_plain` / `Tool(...)` (Pydantic AI) |
| `adk_function_tool`  | `FunctionTool(fn)` wrapping a Python function (Google ADK) |
| `vercel_ai_tool`     | `tool({...})` / `dynamicTool({...})` from the `ai` package |
| `autogen_tool`       | `register_function` / `register_for_llm` / `register_for_execution` (AutoGen, AG2) |
| `unknown`            | Fallback kind; rarely useful                  |

**`scope: agent`** — receives an `AgentDef`; `applies_to` is matched against
`AgentDef.Class` + `AgentDef.SDK`:

| `applies_to` value        | Matches                                                       |
| ------------------------- | ------------------------------------------------------------- |
| `openai_agent`            | `Agent(...)` from `openai-agents` SDK                         |
| `openai_sandbox_agent`    | `SandboxAgent(...)` from `openai-agents` SDK                  |
| `claude_agent_definition` | `AgentDefinition(...)` from `claude-agent-sdk`                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trustabl/agent-reliability-rules](https://github.com/trustabl/agent-reliability-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
