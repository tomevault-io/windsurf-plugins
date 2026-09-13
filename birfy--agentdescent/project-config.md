---
trigger: always_on
description: > **Plugs into [`evolve`](evolution.md) via** `agent=LLMAgent(<completion>)` (or
---

# Connecting agents & LLMs

> **Plugs into [`evolve`](evolution.md) via** `agent=LLMAgent(<completion>)` (or
> `run=`/`propose=`). This page is the completion layer that `LLMAgent` wraps.

`agentdescent.agents` is the **general "talk to a model/agent" layer**. It is
deliberately separate from `agentdescent.evolution` — how you reach a model has
nothing to do with skill evolution, and any application built on the framework
can use it.

The whole contract is one type:

```python
Completion = Callable[[str], str]      # prompt -> text
```

Anything that maps a prompt to text is a completion — an LLM call, a tool-using
agent loop, a canned stub. Adapters build completions; higher layers turn a
completion into whatever task interface they need.

*Module:* [`agentdescent.agents`](https://github.com/Birfy/agentdescent/blob/main/agentdescent/agents.py)
· *API:* [`Completion`, `claude`, `openai_compatible`, `cli_agent`, …](api.md#agents-and-models)
· *See also:* [backends](backends.md) for document tasks, and
[runners](directory-evolution.md) for giving an agent the evolving directory.

## Configuring your provider and key

Credentials are read from the **environment at call time** — they never pass
through code, arguments, or a config file the repo owns. Two variables decide
everything:

| variable | used by | value |
|---|---|---|
| `OPENAI_BASE_URL` | `openai_compatible` | the endpoint's root, e.g. `https://api.deepseek.com` |
| `OPENAI_API_KEY` | `openai_compatible` | your key for that endpoint |
| `ANTHROPIC_API_KEY` | `claude` | your Anthropic key (or run `ant auth login`) |

**DeepSeek**

```bash
export OPENAI_BASE_URL=https://api.deepseek.com
export OPENAI_API_KEY=sk-...
python -m examples.adas.adas_meta_agent_search --provider openai --model deepseek-v4-flash
```

**GLM / Zhipu**

```bash
export OPENAI_BASE_URL=https://open.bigmodel.cn/api/paas/v4
export OPENAI_API_KEY=...
python -m examples.adas.adas_meta_agent_search --provider openai --model glm-4.6
```

**OpenAI** — `OPENAI_BASE_URL` is the default here and may be omitted

```bash
export OPENAI_BASE_URL=https://api.openai.com/v1
export OPENAI_API_KEY=sk-...
python -m examples.adas.adas_meta_agent_search --provider openai --model gpt-4.1-mini
```

**A local server** (vLLM, Ollama, LM Studio) — the key is unused but must be set

```bash
export OPENAI_BASE_URL=http://localhost:8000/v1
export OPENAI_API_KEY=not-used
python -m examples.adas.adas_meta_agent_search --provider openai --model my-local-model
```

**Claude** — a different variable, and `ant auth login` works instead

```bash
export ANTHROPIC_API_KEY=sk-ant-...
python -m examples.adas.adas_meta_agent_search --provider claude --model claude-haiku-4-5
```

Put the `export` lines in your shell profile to keep them across sessions. Every
faithful algorithm port also takes a zero-network `--dry-run`, which prints the
configuration without reading these credentials — the cheapest way to inspect a
run before paying for it:

```bash
python -m examples.adas.adas_meta_agent_search --dry-run
```

!!! tip "Check the endpoint before a long run"
    `--provider openai` talks to whatever `OPENAI_BASE_URL` points at, so a typo
    surfaces as an HTTP error rather than a wrong answer. To see what a key can
    reach:

    ```bash
    curl -s "$OPENAI_BASE_URL/models" -H "Authorization: Bearer $OPENAI_API_KEY"
    ```

    Common replies: `401` — the key is wrong for this base URL; `402` /
    `Insufficient Balance` — the account is out of credit; `404` on
    `/chat/completions` — the base URL is missing or has an extra path segment
    (most gateways want the version prefix, e.g. `/v1`).

## Adapters

```python
from agentdescent import claude, from_callable, echo, with_retries

# Claude (needs: pip install anthropic + credentials / `ant auth login`)
model = claude(model="claude-opus-4-8")           # or claude-haiku-4-5 for cheap runs
text = model("What is 2+2?")

# Any callable you already have
model = from_callable(lambda prompt: my_llm.generate(prompt))

# Deterministic, no-network stub for tests / dry runs
stub = echo(str.upper)          # returns transform(prompt), or the prompt itself

# Wrap any completion with exponential-backoff retries
robust = with_retries(claude(model="claude-haiku-4-5"), attempts=3)
```

`claude()` accepts a `client=` to reuse an existing `anthropic.Anthropic`
instance, and forwards extra kwargs to `messages.create`.

!!! warning "`content: null` is not the same as `""`"
    On OpenAI-compatible endpoints a reasoning model that spends its whole budget
    on `reasoning_content` answers with JSON `null`, not an empty string.
    `openai_compatible` normalises that to `""`, because returning `None` breaks
    the one contract in the package (`Completion` is `prompt -> str`) and used to
    surface as `'NoneType' object has no attribute 'strip'` from inside
    `LLMAgent` — which the engine then **retried as a backend transient**,
    diagnosing a systematic model/parameter mismatch as a flaky endpoint. With the
    empty string the warning above fires instead and names the real cause.

    HTTP errors carry the provider's own message too (`rate limit: retry in 12s`,
    `context length exceeded`), rather than collapsing to `HTTP Error 429`. Extra

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Birfy/agentdescent](https://github.com/Birfy/agentdescent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
