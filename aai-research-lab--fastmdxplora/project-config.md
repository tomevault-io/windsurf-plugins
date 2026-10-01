---
trigger: always_on
description: The FastMDXplora Agent is the natural-language interface. You describe a study
---

# The FastMDXplora Agent

The FastMDXplora Agent is the natural-language interface. You describe a study
in a sentence; it writes a [FastMDXplora Config](config.md).

```bash
fastmdx agent "simulate trypsin with benzamidine bound at pH 6.5 for 100 ns"
```

```yaml
systems:
  - system: 3PTB
setup:
  ph: 6.5
  forcefield: amber-openff
  ligand_name: BEN
simulation:
  duration_ns: 100
agent: assisted
agent_model: anthropic/claude-sonnet-4-6
```

That Config then runs like any other:

```bash
fastmdx explore --config study.yml
```

The Agent stands where a human stands. It is a fourth way of producing a
Config, alongside the [GUI](gui.md), the [CLI](cli.md) and the
[API](api.md) — and it is reachable **through all three of them**.

---

## The Agent has no privileges

This is the load-bearing claim, so it is worth stating plainly.

**A Config the Agent writes goes through exactly the same validator, by the
same code, as a Config you type by hand.** There is no separate path, no
relaxed mode, no special case. `fastmdxplora.agent` imports the core; the core
imports nothing from the Agent, and a test asserts the direction.

```python
# fastmdxplora/agent/propose.py
from fastmdxplora.config.loader import ConfigError, validate_config
...
try:
    validate_config(config)
except ConfigError as exc:
    ...
```

Three consequences:

- **A proposal that did not validate carries no Config at all** — not the last
  thing that nearly worked with a caveat attached. There is no partially-valid
  result and no best-effort fallback.
- **The Agent cannot name a setting that does not exist**, because the schema
  description it writes from is generated from the same declaration the
  validator checks against.
- **Removing the Agent changes nothing about what a valid study is.** A caller
  bypassing it and calling `validate_config` directly is refused in the same
  way, by the same code, with the same message.

The one thing an Agent-written study carries that a hand-written one does not
is the `agent:` setting, which is **provenance, not permission**. Including
`agent: unvalidated` — see below — the Config itself is still validated.

---

## Connecting a model

Nothing in FastMDXplora needs a model. The Agent does, and it asks once.

```
$ fastmdx agent set
  Model:
    [1] Anthropic
    [2] OpenAI
    [3] Other (any OpenAI-compatible URL)
  > 1
  Model [claude-sonnet-4-6]:
  API key (leave blank to read ANTHROPIC_API_KEY from the environment instead):
  > sk-ant-...

  ✓ Saved to ~/.config/fastmdxplora/model.json
  ✓ Key stored there, readable only by you. It is never written into a study.
```

| Provider | Default model | Environment variable |
|---|---|---|
| `anthropic` | `claude-sonnet-4-6` | `ANTHROPIC_API_KEY` |
| `openai` | `gpt-5` | `OPENAI_API_KEY` |
| `compatible` | whatever you name | `FASTMDX_MODEL_API_KEY` |

Option 3 covers DeepSeek, vLLM, Ollama, OpenRouter and most local servers,
because they speak the OpenAI chat shape. One entry rather than one per vendor:
a list of vendors goes stale and a protocol does not. It takes a base URL —
`https://api.deepseek.com`, `http://localhost:11434/v1`,
`http://localhost:8000/v1`.

**Nothing extra has to be installed.** `fastmdxplora.agent` ships with the
package; `pip install "fastmdxplora[agent]"` installs no additional
dependencies. The Agent talks to a model over the standard library, with no
vendor client library anywhere in it.

### Where the key lives

In one file outside any study, readable only by its owner — or in the
environment, **which is checked first**. That is how a cluster job or a CI run
supplies one without anybody storing it.

| | |
|---|---|
| `$FASTMDXPLORA_CONFIG_DIR/model.json` | if that variable is set |
| `$XDG_CONFIG_HOME/fastmdxplora/model.json` | otherwise, defaulting to `~/.config/` |
| `%APPDATA%\fastmdxplora\model.json` | on Windows |

Written with mode `0600`.

**The key never enters a Config, a Manifest, a log line or an error message.**
Those files get shared, pasted into issues and committed; a key in one is a key
on the internet. What is recorded is the provider and the model and nothing
else.

---

## The three modes

```bash
fastmdx agent "..." --assisted        # the default
fastmdx agent "..." --autonomous --budget-hours 40
fastmdx agent "..." --unvalidated
```

They are mutually exclusive, and each writes itself into the Config as
`agent: <mode>`. The budget writes itself in too, as `budget_hours`, a
top-level key with a floor of zero. It is read by `explore` whichever door
the Config came through: a budgeted Config runs in stages, setup first, then
a price, then the rest only if it fits. Required for `autonomous`, which runs
without being shown to anybody; optional in every other mode, and never wrong
to set on a study that will run for days.

### `assisted` — draft it and stop

The default. The Config prints, the repair attempts print with it, and `-o`
also writes it to a file. Nothing runs. You are there to read it.

```bash
fastmdx agent "simulate ubiquitin at pH 6.5 for 50 ns" -o ubiquitin.yml
fastmdx explore --config ubiquitin.yml
```

### `autonomous` — draft it and run it

Nobody is there to read it, so something else has to stop it, and that is a
budget.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aai-research-lab/FastMDXplora](https://github.com/aai-research-lab/FastMDXplora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
