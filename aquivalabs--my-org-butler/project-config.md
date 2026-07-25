---
trigger: always_on
description: This repo is mid-migration from classic Agentforce metadata to **Agent Script**.
---

# My Org Butler — Agent Script Migration

This repo is mid-migration from classic Agentforce metadata to **Agent Script**.
Branch: `migration-to-agent-script-v2`. This file describes the current state and plan —
nothing else. Coding standards live in `.claude/rules/`.

## Architecture (current)

- **The agent** is `unpackaged/main/default/aiAuthoringBundles/MyOrgButler/MyOrgButler.agent`
  (AiAuthoringBundle). One subagent (`butler`), 18 actions: 15 Apex (`apex://<Class>`),
  1 prompt template (`generatePromptResponse://AnswerFromFile`), 2 standard actions
  (`EmployeeCopilot__IdentifyRecordByName`, `EmployeeCopilot__GetRecordDetails`).
  Router (`agent_router`) transitions unconditionally to the butler via `after_reasoning`;
  the butler runs on Claude Sonnet 4.6 (`model_config`), the org default model is
  AWS-hosted Anthropic (Setup → Einstein Audit, Analytics, and Monitoring Setup).
  Butler reasoning carries anti-fabrication rules (never claim success without a
  successful action call, explicit tool mentions MUST call the tool, schema answers
  only from ExploreOrgSchema).
- **Classic agent metadata** is deleted from the working tree (also the classic Bot/
  GenAiPlannerBundle copies that lived in `unpackaged/` — they broke fresh-org deploys
  once the plugins were gone). Git tag `pre-agent-script-migration` marks the last
  commit with the full classic implementation.
- Deploy → publish → activate: deploying the bundle only stores source in the org.
  `sf agent publish authoring-bundle --api-name MyOrgButler --skip-retrieve` compiles it
  into Bot/BotVersion/GenAiPlannerBundle; activation makes it runnable.
  `scripts/create-scratch-org.sh` does all of this.

## Remaining phases

1. **Deterministic refactor** (next): replace prompt hacks with language constructs —

   | Classic hack | Agent Script construct |
   |---|---|
   | "LoadCustomInstructions MUST be FIRST action" | `run @actions.LoadCustomInstructions` + `set @variables.custom_instructions` before reasoning |
   | Planner attributeMappings | `set @variables.… = @outputs.…` on load AND store |
   | "ALWAYS ExploreOrgSchema/ExploreDataCloud FIRST" | `available when @variables.…_explored == True` gates |
   | "NOT for: …" negative routing in descriptions | subagent split (`data`, `data_cloud`, `org_dev`, `files_web`, `memory`, `automation`) + router |
   | Headless "NEVER create another plan" rules | dedicated `automation` subagent, visible only after transition |

   Known agent bugs this must fix (from test runs): claims "Noted" without calling
   StoreCustomInstruction; answers memory questions without calling LoadCustomInstructions;
   doesn't surface memory content.
   Syntax authority: `trailheadapps/agent-script-recipes` → `.airules/AGENT_SCRIPT.md`.
2. **Packaging**: AiAuthoringBundle and AiTestingDefinition are NOT 2GP-packageable
   (confirmed against the 2GP Agentforce packaging doc). Plan: ship the `.agent` (and
   optionally the test XMLs) as plain-text static resources — admins open
   `<org-url>/resource/AgentScript` in the browser, copy, paste into Agent
   Studio. No LWC viewer (decided against vendoring forcedotcom/lwc-agentscript-viewer).
   Namespace caveat: the packaged copy needs `apex://aquiva_os__…` targets (and
   `aquiva_os__AnswerFromFile`); the inject transform does NOT exist yet — only the
   strip in create-scratch-org.sh.
3. **README** showcase: "one agent, two implementations".

## Testing

Test definitions live in `unpackaged/main/default/aiTestingDefinitions/`, eval configs
in `scripts/`. All on the **Agentforce Studio (NGT) test runner**
(Beta — the legacy AiEvaluationDefinition flow is officially "legacy" and was deleted here):

- `AgentRegression.aiTestingDefinition-meta.xml` — 14
  per-action cases. Run: `sf agent test run --api-name AgentRegression --wait 30`.
  Every case (except the dedicated LoadCustomInstructions test) presets the
  `custom_instructions` context variable so the router's deterministic load is skipped
  and the case asserts ONLY its own action. `action_sequence_match` is strict — stray
  planner action calls fail it.
- `PromptRegression.aiTestingDefinition-meta.xml` —
  prompt-template smoke tests (ConsolidateMemory via conversationHistory, AnswerFromFile).
- `scripts/demo-story.yaml` — the multi-turn conference-demo conversation, driven over the REST
  endpoint below, judged by Claude in-memory. Covers what single-turn can't.

NGT format facts: testCases have `inputs:` (utterance + optional contextVariables /
conversationHistory — **every** history turn needs a `topic`, even user turns) and
`scorers:` (topic_sequence_match, action_sequence_match, agent_handoff_match,
bot_response_rating, response_match, coherence, conciseness, factuality, completeness,
task_resolution, output_latency_milliseconds). Label/description max 80 chars.
The CLI returns actual/expected pairs but **no pass/fail verdicts** — grade yourself or
in the Studio UI. YAML specs are transient: `sf agent generate test-spec
--from-definition <xml>` ⇄ `sf agent test create --spec <yaml> --test-runner
agentforce-studio`. Repo keeps only the XML.

Smoke-test channel (works for classic and script agents):

    POST /services/data/v66.0/actions/custom/generateAiAgentResponse/MyOrgButler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aquivalabs/my-org-butler](https://github.com/aquivalabs/my-org-butler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
