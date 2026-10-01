---
trigger: always_on
description: - Backend `type`: `AGENT`
---

### Agent

- Backend `type`: `AGENT`
- Default node subtype: always use private agent mode by setting metadata/spec `type` to `PRIVATE`.
- Do not create/use an `AGENT` node when no tools are attached. If the behavior is pure prompting/reasoning with no attached tools, use an `LLM` node instead.
- Only use a reusable/external agent selection flow when the user explicitly asks to use an existing reusable agent.

#### Structure

- A valid `AGENT` node must include:
  - `metadata.type`
  - an outer `outcomes.success`
  - an input named `message`
- `AGENT` uses the normal forward `success` path.
- Do not invent a normal `failure` outcome for `AGENT`. If error handling is needed, use the shared `metadata.errorNodeId` pattern.
- `metadata.type` must be exactly `PRIVATE` or `REUSABLE`.
- `message` is always a normal node input. In the workflow spec it should be stored as `type: "string"` with a string `value`.
- When using `do-create-node` or `do-modify-node`, set `inputsPatch.message` to the raw message string only.
- Do not pass nested input objects such as `{ "type": "string", "value": "..." }` for `message`.
- Do not pass full input objects such as `{ "id": "...", "name": "message", "type": "string", "value": "..." }`.

#### Private Agent Properties

- **Inputs:**
  - `prompt` (required; text/template; supports expressions)
  - `message` (required; text/template; supports expressions)
- **Metadata / specification fields:**
  - `type`: `PRIVATE` (required for nodes you create)
  - `modelConfiguration` (optional) — same model settings surface as LLM node
  - `chatHistoryEnabled` (boolean)
  - `answerInUserLanguage` (boolean)
  - `maxInteractions` (required number)
  - `agentRole` (optional string; persona/instructions)
  - `summarizationMode` (`Default` | `Custom` | `Disabled`)
  - `summarizationPrompt` (string; only meaningful when `summarizationMode` is `Custom`)
  - `toolCodes` (string[]; attached tools)
  - `topicCodes` (string[]; optional attached topics)
  - `aiAppOutputSpecification.dataDisplay` (optional object for App Experience settings)
  - `outputSpecification` (optional JSON schema/string for expected output)
- Note: the editor reuses the LLM model selection fields and also exposes app-experience fields on Agent nodes.
- For private agents, `prompt` is a normal node input and should be stored as `type: "string"` with a string `value`.
- When using `do-create-node` or `do-modify-node`, set `inputsPatch.prompt` to the raw prompt string only.
- Do not pass nested input objects or full input-entry objects for `prompt`.
- For private agents, `maxInteractions`, `agentRole`, `summarizationMode`, `summarizationPrompt`, and private `outputSpecification` belong under `metadata.specification`, not as top-level metadata fields.
- For private agents, `toolCodes` and `topicCodes` are persisted as top-level metadata arrays on the node, not inside `metadata.specification`.
- `maxInteractions` is required for private agents and should be an integer from `1` to `20`.
- If `summarizationMode` is `Disabled`, persist an empty `summarizationPrompt`.
- If `summarizationMode` is `Default`, persist the default summarization mode and prompt instead of inventing a custom prompt value.

#### Reusable Agent Properties

- Use reusable mode only when the user explicitly wants an existing reusable agent.
- Reusable agents must set `metadata.type = REUSABLE`.
- Reusable agents must set `metadata.agentCode` to the selected reusable agent's real code.
- Reusable agents still require the normal string-backed `message` input.
- Additional reusable-agent variable inputs may exist beyond `message`. Those inputs must come from the selected reusable agent's published specification.
- Do not invent adhoc reusable-agent variable inputs that are not declared by the selected reusable agent.
- Preserve the declared type of each reusable-agent variable input from the selected agent specification.
- If the selected reusable agent changes, stale variable inputs from the previous reusable agent must not remain on the node.
- Reusable agents may expose tools, topics, and a read-only output schema from the selected reusable agent definition. Treat those as selection-derived behavior, not something to improvise freely.

#### Chat history

- If the Agent needs prior chat turns, set `metadata.chatHistoryEnabled = true`.
- When using `do-create-node` or `do-modify-node`, set this through `metadataPatch.chatHistoryEnabled = true`, not `inputsPatch`.
- Do not inject chat history into Agent `message`, `prompt`, `summarizationPrompt`, or Code node JavaScript with `$context.$system.$chatHistory`.
- Do not invent `$context.$system.$conversationHistory`.
- Use `{{$context.$system.$inputMessage}}` for the current user message when needed; let `chatHistoryEnabled` supply prior turns.

#### Concrete Examples

Valid private-agent shape:

- `CREATE_DRAFT_WITH_TOOLS.type = AGENT`
- `CREATE_DRAFT_WITH_TOOLS.metadata.type = PRIVATE`
- `CREATE_DRAFT_WITH_TOOLS.metadata.chatHistoryEnabled = true`
- `CREATE_DRAFT_WITH_TOOLS.outcomes.success = REVIEW_OUTPUT`
- `CREATE_DRAFT_WITH_TOOLS.inputs.message.type = string`
- `CREATE_DRAFT_WITH_TOOLS.inputs.message.value = "Draft an announcement for {{$context.$nodes.LOAD_INPUT.$output.topic}}."`
- `CREATE_DRAFT_WITH_TOOLS.inputs.prompt.type = string`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oracle/fusion-ai-studio](https://github.com/oracle/fusion-ai-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
