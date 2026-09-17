---
trigger: always_on
description: Conventions for AI agents and contributors in this repository. Keep it
---

# AGENTS.md

Conventions for AI agents and contributors in this repository. Keep it
factual: paths, commands, invariants — no essays.

## Project layout

- `lib/` — the `flutter_agent_harness` package (pure Dart core). `test/`
  mirrors it. `prompts/` — all LLM prompts as Markdown (see rules below).
- `lib/src/approval/` — tool approval gate: tiers (read/write/exec),
  session modes (always-ask/write/yolo/unattended), per-tool overrides,
  critical-pattern
  `bash` interceptor, per-turn grants (`grantForTurn(allow:, deny:)` —
  skill `allowed-tools`/`disallowed-tools` manifest keys ride these for one
  turn; explicit deny > turnDeny > per-tool override > turnAllow >
  alwaysAllow > mode (critical bash outranks mode in always-ask/write/yolo,
  but is skipped in unattended so the mode never blocks — for runs with no
  user present); `clearTurnGrants()`
  on every new user
  message). Wired via `attachApproval` into `beforeToolCall` (runs
  first); prompt UI is an injectable `ApprovalPrompt` (null + prompt policy
  = deny).
- `lib/src/tools/ask_tool.dart` — `ask` tool: structured mid-turn questions
  via injectable `AskCallback` (null = error, cancel = plain result).
- `lib/src/tools/request_secret_tool.dart` — `request_secret` tool: the agent
  asks the USER for a missing credential via injectable
  `RequestSecretCallback` (never in chat text); a grant returns
  `RequestSecretResult` (host-adjusted `name`, `value`, `persisted` flag),
  a decline is a plain result. The app wires it in `AgentService` to
  `secretRequestHandler` (chat screen installs
  `ui/widgets/secret_request_sheet.dart`): a grant is persisted into
  `SessionKeysStore`, injected into the live shell env via
  `SecretsExecutionEnv.addSecrets` (the map is runtime-mutable now), and
  registered into the same `SecretRedactor` — so the next run's prompt name
  list, bash `$NAME`, and redaction all pick it up.
- `lib/src/env/session_vars_execution_env.dart` — `SessionVarsExecutionEnv`:
  an `ExecutionEnv` decorator injecting session-correlation env vars
  (`FAH_SESSION_ID`/`FAH_SESSION_FILE`/`FAH_PROVIDER`/`FAH_MODEL`, resolved
  live per `exec`, never secrets) into bash tool executions. Wired around
  `builtinTools` in the CLI (`AgentCli`) and the app (`AgentService`).
  `LocalShell` merges `ShellExecOptions.env` OVER `Platform.environment`
  (never replaces), so injected vars keep the inherited environment.
- `lib/src/tools/checkpoint_tool.dart` — `checkpoint`/`rewind` tools:
  context hygiene for detours. `CheckpointRewindController` wraps
  `Agent.prepareNextTurn`, persists via host `CheckpointSessionSink`.
  Lifecycle (issue #286): a checkpoint is scoped to its detour — when the
  next `checkpoint` call finds it stale (a user turn arrived inside the
  detour, or the anchored span was rebuilt away by reload/compaction), it
  is auto-closed with a note + `checkpoint_auto_closed` session record
  instead of refusing; `rewind` semantics are unchanged.
- `lib/src/compaction/branch_summarization.dart` — `generateBranchSummary` +
  `navigateSessionTree` (use instead of `Session.moveTo` for tree
  navigation); summary is a `branch_summary` record on the entered branch.
- `lib/src/agent/tool_pairing.dart` — context pair-integrity (issue #85):
  `validateToolPairing` checks the wire-equivalent message sequence (orphan
  results, unanswered calls, duplicate ids, displaced results after
  same-role merges); `repairToolPairing` fixes the outbound payload only
  (drop orphan + note, synthesize interrupted results, hoist steering text
  after results, uniquify duplicate ids symmetrically on call+result).
  `agent_loop.dart` repairs before every request and retries once on a
  pairing-shaped provider 400 (`ToolPairingRepairEvent` is the audit
  trail); `_finishStreamed` stamps per-run position-counter ids
  session-unique; `auto_compactor.dart`'s `_localTrimFallback` routes the
  kept region through the same repairer.
- `lib/src/agent/image_registry.dart` — session image registry (issue
  #171): unique images ride a provider request exactly once; every other
  occurrence becomes a `[Image N]` text ref. `rewriteHistoryImages`
  rewrites the OUTBOUND payload only (session JSONL byte-identical);
  carriers `[{text:"[Image N]"},{image}]` anchor before the first
  referencing user message or after the tool-result run (never inside
  call/result pairs); the current user message rides images in place
  (I3), each in-place image carrying its `[Image N]` label (issue #195
  F3 — the model can cite what it sees); the per-request cap
  (`images.maxPerRequest`, default 20) drops current-first-then-newest
  with a drop notice (never silent; the app logs drops via `AppLog`,
  issue #195 F4); dangling refs (compaction, cap) resolve to `(image no
  longer available)`, and once a renumbering boundary exists (compaction
  summary, structured marker, local-trim note) AUTHORED history
  citations degrade to that note too — after renumbering an `[Image N]`
  may name a different image, so silent rebinding is never allowed
  (issue #195 F2; generated refs/labels stay content-keyed and fresh).
  Stateless per-request rebuild — determinism, compaction eviction and
  resume come free. Kill switch `images.registry: false` → byte-for-byte

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IstiN/flutter_agent_harness](https://github.com/IstiN/flutter_agent_harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
