---
trigger: always_on
description: A turn-based agent framework. The runtime is the point: the loop, the event
---

# CLAUDE.md

A turn-based agent framework. The runtime is the point: the loop, the event
protocol, and the two projections (`parts` for storage, `context_text` for
the model) are ours; provider SDKs are quarantined in `providers/`.

## Layout

```text
src/void_agent/core/     the concepts, finely split (one concern per file):
                         leaves errors, content (text/image/PDF inputs), messages, ask (a question as a
                         value), usage (what a round-trip cost as a value);
                         packages human/ (attendant: who answers —
                         the protocol, the ambient contextvar, and the two
                         ready-made ones: `ScriptedHuman` and
                         `HumanChannel`, paired with a queue of `Question`s
                         replied to in place; channel: how a question
                         travels, `Unanswered`),
                         tool/ (tool: the mechanism; gate: decides in code,
                         asks the person itself), agent/ (agent, loop,
                         dispatch, outcome, rules), builtins/
                         (plan, reflection — shipped tools, chain-enabled
                         via `with_plan`/`with_reflection`, never injected),
                         events/ parts/ llm/ — pydantic only
src/void_agent/providers anthropic.py / openai.py — SDK types stay inside
src/void_agent/mcp       the bridge: an MCP server's tools as `Tool`s
                         (server.py mounts and discovers, result.py maps a
                         CallToolResult); the approval is declared by the
                         caller, never by the server; SDK types stay inside
src/void_agent/skills    a folder of instructions as tools: only the
                         description stays in context, the body arrives on
                         the call; knowledge, never capability
src/void_agent/__init__  the public API; users import from `void_agent`
cli/                     the terminal UI, in-process (repo-only, not
                         packaged): the `universal` agent — the model, a
                         plan, reflection, ask_user, void's own `toolbox`
                         server (read, search, edit, run over one root),
                         and whatever `~/.void/mcp.json` and
                         `~/.void/skills` mounted, marked in `/mcp` and
                         `/skill`; `weather` (cli/agents/weather.py: Open-Meteo
                         behind five thin tools, the model as the
                         scheduler) and `dummy_weather` (the same on a
                         scripted model and canned data); the agents
                         scanned, never registered — the built-in shelf,
                         `~/.void/agents`, each `--workspace` folder —
                         listed in `/agent`, each reaching the others by
                         name through the pool;
                         the protocol rendered from parts, cards
                         answered in place, attachments as `file` parts;
                         the model from Anthropic, OpenAI, or a local
                         Ollama (its OpenAI-compatible `/v1`, the installed
                         list read live); `make cli` runs it, `make
                         cli-build` packs dist/void; see cli/CLAUDE.md
scripts/                 compile.py — the native wheel: core/ + providers/ as
                         .so, stubs beside them (`make compile`; its
                         docstring is the guide); pack.py — the CLI as one
                         binary (`make cli-build`); smoke_toolbox.py — one
                         MCP round trip against a packed binary;
                         screenshots.py — the README's screenshots, drawn
                         by the app on a live model (a key in the
                         environment; nothing of it reaches the file)
screenshots/             what scripts/screenshots.py drew; committed, since
                         the README shows them
tests/                   behavior-named, ScriptedLlm as the seam
```

Dependencies point one way inside core: `messages → content`,
`llm → content/events/errors/usage`, `agent → everything below`,
`builtins → tool/events`, `tool → ask/errors/events/human`,
`human → ask/events`, `events → ask/usage`, `parts → events/usage`; then `providers → core`,
`mcp → core`, `skills → core`; then `cli → void_agent`. Concept modules
never import providers, and the framework never imports the CLI: a client
that mounts an agent is an application above it.

## Invariants (do not break)

- `final_answer` is the output submission protocol, not a registered Tool.
  `.output(Model, instructions=...)` adds agent guidance to its description;
  guidance never replaces validation or the completion rules.
- A valid `final_answer` preempts BEFORE side effects: no sibling call
  runs, no transcript growth. `ask_user` is put to the attendant before
  any sibling side effect: answered, the answer is the call's result and
  the step goes on; unanswered, the turn ends there with the card open.
- The person attends the run through one channel (`run(..., human=…)`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoyipyik/void-agent](https://github.com/hoyipyik/void-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
