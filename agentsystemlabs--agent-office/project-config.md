---
trigger: always_on
description: Back to the [README](../README.md).
---

# Choosing an agent

Back to the [README](../README.md).

Every new worker starts on the office's **Default worker**: a provider, model and effort an admin picks in ⚙️ Settings → **🤖 Workers** (the `--agent` with its own default model until someone does). The hire, ask, queue, issue and meeting windows show it on one line; click **✏️ Edit** there to pick **Claude Code**, **OpenCode**, **Codex**, **Grok**, **Muse**, **DeepSeek Harness**, **Pi** or **Cursor**, a model and an effort for just that worker or task, and **↺ Use the default** to go back. Board agents and tasks the Queue agent adds, which nobody picks for, start on the default too. Existing workers keep their provider when prompted or resumed, and queued tasks keep their choice when retried or restored after a restart.

Every harness takes both a **Model** and an **Effort**, each optional: leave one on **Default** and the harness uses its own settings.

| Harness | Model | Effort |
| --- | --- | --- |
| Claude Code | Fable 5.1, Opus 5.5, Sonnet 5.5 or Haiku 4.5 (`--model`) | `--effort` |
| OpenCode | a `provider/model`, typed or picked from the suggestions `opencode models` gives (`--model`) | the model's variant |
| Codex | one of the models `codex debug models` lists (`--model`) | `-c model_reasoning_effort` |
| Grok | one of the models `grok models` lists (`--model`) | `--effort` |
| Muse | a model id, typed (`--model`) | `--reasoning-effort` |
| DeepSeek Harness | a model id from its catalog, typed | its `reasoning_effort` option |
| Pi | a model name or `provider/model`, typed (`--model`) | **Thinking** (`--thinking`) |

The efforts are low, medium, high, extra high and max. Where a harness's own catalog says which of them a model supports (Codex and OpenCode do), the ones it doesn't are greyed out once that model is picked. A harness whose models can't be listed (its CLI isn't installed, or isn't signed in) still takes a model id typed in. A **Custom** `--agent` is run as it is, so it has neither field: set its model and effort in `--agent-args`.

What's picked shows next to the worker everywhere it's named (the Workers panel, its task card, its terminal's title, the queue), by the model's name rather than its family: **Opus 5.5**, not "Opus". For Claude Code that name comes from the session itself once the worker has spoken, so a worker left on Default says which model the default is, and one switched with `/model` in its terminal says the new one.

For OpenCode, the **Model** field selects the initial model for a new worker or queue task. Suggestions come from `opencode models`; you can also enter a `provider/model` ID. Leave it empty to use your OpenCode settings (including configured CLI arguments). An explicit choice overrides configured model arguments for that launch and stays with a queued task when retried. **Effort** is what OpenCode calls the model's *variant*. Its TUI has no flag for one, and sends each message at the variant it remembers for the model from whichever session used it last, so the office's plugin puts the worker's first message on the effort picked for it; the TUI carries on from that message, and `Ctrl+T` in the terminal still changes it. An effort the model has no variant for changes nothing.

**Pi** is available in the worker picker, queued tasks, meetings, and the office default. Install and sign in to the Pi coding-agent CLI on the machine running the office (`npm install -g @earendil-works/pi-coding-agent`, then `pi` and `/login`). The integration targets Pi 0.87.1 or later, using its `agent_settled` and UI prompt extension events. Choose Pi when hiring, or start the office with `--agent pi`. Windows npm `.cmd` launchers are supported.

Pi keeps its own login, settings, packages, and extensions. The office adds a CLI-loaded extension to report session, working, tool, waiting-for-input, and completion events; it does not edit your Pi configuration. Each worker's sessions live under `.agent-office/pi-sessions/<worker-id>`, and **R** resumes that worker's session. The optional **Model** field accepts a model name or `provider/model`; **Thinking** offers low, medium, high, xhigh, and max. Leave either on Default to use Pi settings or configured `--agent-args` (including `--thinking off` or `minimal`). Cost and token metrics remain in Pi's terminal and are not included in the office's Claude budget.

For Claude Code, the **Model** and **Effort** fields pick `--model` (`fable`, `opus`, `sonnet` or `haiku`, listed by the model each alias stands for: Fable 5.1, Opus 5.5, Sonnet 5.5, Haiku 4.5) and `--effort` (low, medium, high, extra high or max) for that worker or queue task, overriding whatever `--agent-args` set office-wide. Leave either on **Default** to use the office's configured value. The choice stays with a worker across resumes and with a queued task when retried, and shows up next to the worker in the Workers panel and on its task card so you can tell at a glance who's on what — and the per-worker cost there reflects the actual model used.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentSystemLabs/agent-office](https://github.com/AgentSystemLabs/agent-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
