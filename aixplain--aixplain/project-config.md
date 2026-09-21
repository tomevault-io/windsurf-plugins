---
trigger: always_on
description: Validates conversation history for agent sessions.
---


Agent module for aiXplain v2 SDK.

### ConversationMessage Objects

```python
class ConversationMessage(TypedDict)
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L45)

Type definition for a conversation message in agent history.

**Attributes**:

- `role` - The role of the message sender, either &#x27;user&#x27; or &#x27;assistant&#x27;
- `content` - The text content of the message
- `attachments` - Optional attachments — hosted-URL/local-path strings or dicts
  with ``url`` or ``path`` (plus optional type/name/mimeType).
- `files` - Deprecated. Local file paths to upload — pass through ``attachments``.

#### validate\_history

```python
def validate_history(history: List[Dict[str, Any]]) -> bool
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L62)

Validates conversation history for agent sessions.

This function ensures that the history is properly formatted for agent conversations,
with each message containing the required &#x27;role&#x27; and &#x27;content&#x27; fields and proper types.

**Arguments**:

- `history` - List of message dictionaries to validate
  

**Returns**:

- `bool` - True if validation passes
  

**Raises**:

- `ValueError` - If validation fails with detailed error messages
  

**Example**:

  &gt;&gt;&gt; history = [
  ...     \{&quot;role&quot;: &quot;user&quot;, &quot;content&quot;: &quot;Hello&quot;},
  ...     \{&quot;role&quot;: &quot;assistant&quot;, &quot;content&quot;: &quot;Hi there!&quot;}
  ... ]
  &gt;&gt;&gt; validate_history(history)  # Returns True

### OutputFormat Objects

```python
class OutputFormat(str, Enum)
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L120)

Output format options for agent responses.

### ContextOverflowStrategy Objects

```python
class ContextOverflowStrategy(str, Enum)
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L128)

Strategy applied when input messages exceed the model&#x27;s context window.

**Attributes**:

- `TRUNCATE` - Remove the oldest chat-history messages until the context fits.
- `SUMMARIZE` - Replace the full chat history with an LLM-generated summary.

### AgentRunParams Objects

```python
class AgentRunParams(BaseRunParams)
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L230)

Parameters for running an agent.

**Attributes**:

- `session` - Conversation thread to run within. A
  :class:`~aixplain.v2.session.Session` instance or a session id
  string. Omit for a one-shot, stateless run. Replaces the removed
  ``via_session`` flag and id-only ``session_id``.
- `query` - The query to run
- `variables` - Variables to replace \{\{variable}} placeholders in instructions and description.
  The backend performs the actual substitution.
- `tasks` - List of tasks for the agent
- `prompt` - Custom prompt override
- `~aixplain.v2.session.Session`0 - Conversation history
- `~aixplain.v2.session.Session`1 - Execution parameters (maxTokens, etc.). Passing
  ``max_iterations`` here is deprecated; set ``agent.budget.max_iterations``
  instead. A deprecated value is folded into ``budget.max_iterations``
  (the agent&#x27;s budget wins on conflict) and the standalone key is not
  emitted.
- `~aixplain.v2.session.Session`8 - Criteria for evaluation
- `~aixplain.v2.session.Session`9 - Evolution parameters
- ``0 - Inspector configurations
- ``1 - Whether to run response generation. Defaults to False.
- ``2 - Multimodal attachments for the turn.
  Each entry is a hosted-URL/local-path string or a dict with ``url`` or
  ``path`` (plus optional ``type``/``name``/``mimeType``). Local paths are
  uploaded to aiXplain storage automatically.
- ``3 - Deprecated. Local file paths to upload — pass through ``attachments`` instead.
- ``6 - Display format - &quot;status&quot; (single line) or &quot;logs&quot; (timeline).
  If None (default), progress tracking is disabled.
- ``7 - Detail level - 1 (minimal), 2 (thoughts), 3 (full I/O)
- ``8 - Whether to truncate long text in progress display

### Budget Objects

```python
@dataclass_json

@dataclass
class Budget()
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L285)

Budget caps governing an agent run (cost / duration / iterations).

Every :class:`Agent` owns a ``budget`` (defaulting to an empty ``Budget()``),
mutated in place via attribute access — mirroring ``model.inputs``::

    agent.budget.max_cost = 0.5
    agent.budget.max_iterations = 10

The same object serves two roles: ``agent.save()`` persists it as the agent&#x27;s
default budget, and ``agent.run(...)`` sends its current state as the run-time
budget (the backend merges the run-time budget field-by-field over the
persisted default). The Python API is snake_case; serialization produces the
agreed camelCase wire keys (``maxCost`` / ``maxDurationSeconds`` /
``maxIterations``). All fields are optional and ``None`` fields are dropped
from ``to_dict()``.

### AgentResponseData Objects

```python
@dataclass_json

@dataclass
class AgentResponseData()
```

[[view_source]](https://github.com/aixplain/aiXplain/blob/main/aixplain/v2/agent.py#L319)

Data structure for agent response.

#### \_\_post\_init\_\_


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aixplain/aiXplain](https://github.com/aixplain/aiXplain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
