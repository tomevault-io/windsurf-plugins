---
trigger: always_on
description: This file is for **coding agents**. Read it (not the whole codebase) to author new
---

# AGENTS.md — building flows in `saage`

This file is for **coding agents**. Read it (not the whole codebase) to author new
flows and skills for this engine. It is complete enough to build a working flow
from scratch; drop into the source only when you need an internal detail.

> Building a flow from a user request? Follow the step-by-step recipe in
> [`.claude/skills/building-saage-flows/SKILL.md`](.claude/skills/building-saage-flows/SKILL.md).
> The request still vague? Interview the user first with
> [`.claude/skills/designing-saage-flows/SKILL.md`](.claude/skills/designing-saage-flows/SKILL.md).
> Despite the directory name both are harness-agnostic — plain markdown any agent
> (Codex, Gemini, Copilot, …) can follow; the frontmatter is just discovery
> metadata for harnesses with skill support. To *run* flows as native tool
> calls instead of shelling out, connect to the `saage mcp` server
> ([docs/agents.md](docs/agents.md)).

## Mental model (read this first)

`saage` is a **deterministic** workflow engine. *Control flow* — loops, retries,
polling, exit conditions, ordering — is owned by **code/YAML**, never by an LLM's
judgment. LLMs are used only **inside a step** to produce *content* (write code,
write SQL, review a diff, summarize). So when you design a flow you are deciding:

- which steps are **deterministic** (`command`) vs **LLM** (`agent`), and
- how steps are wired with the three **loop primitives**.

A run shares one mutable dict, the **shared store**. Steps read inputs from it via
`{{ templates }}`, and write outputs back into it via `set:` regex captures. All
loop state lives in the shared store (the engine shallow-copies nodes each step,
so nothing else persists).

## Repo map

| Path | What |
|---|---|
| `flows/<name>/flow.yaml` | a flow: provider + shared seed + the `workflow` step list |
| `flows/<name>/<skill>/skill.md` | one skill = frontmatter + instruction body |
| `flows/<name>/<skill>/*.py` | optional helper scripts a step runs (see *Helpers*) |
| `saage/hydrate.py` | YAML → runnable flow (the schema authority) |
| `saage/nodes.py` | `AgentNode`, `CommandNode`, `render()`, loop guards |
| `saage/primitives.py` | `retry_loop`, `polling_loop`, `counting_loop` |
| `saage/tools.py` | the harness tools (file CRUD, `run_command`, git) |
| `saage/skills.py` | how `skill.md` is parsed |
| `saage/server/` | FastAPI web UI + REST API for job management (`pip install saage[server]`) |

Existing flows are the best templates: `story_writer` (counting_loop),
`fix_failing_test` (retry_loop), `poll_job` (polling_loop), `guessing_game`
(counting_loop + `exit_when` + shared feedback), `greenfield_ml` (everything).

### `saage/server/` — job manager web UI

The server is a **thin layer over the run-store**: it lists flows from `flow_paths`, launches
detached `saage run` subprocesses, and tails their ledgers and logs. The web UI (home, job detail,
history pages) and REST API (`/api/flows`, `/api/jobs`, `/api/parse`) poll the run-store at
`~/.saage/runs/<job_id>/` — no database or job queue. Each running job emits ledger events
(`ledger.jsonl`, one JSON record per line) with `phase: "start"` / `phase: "end"` markers per node
that the DAG visualizer consumes to render live progress. To run server tests:

```bash
pytest tests/server/ -q
```

Docs: See the "Web UI: `saage serve`" section in the main README.

## `flow.yaml` reference

```yaml
provider: { type: openrouter, model: "deepseek/deepseek-v4-flash" }   # optional pin
workspace: /tmp/saage_run        # optional: tool/command cwd. default = the flow dir
venv: .venv                    # optional: auto-activated for commands once it exists
artifacts: [experiments.jsonl, "report*.html"]   # optional: workspace files/globs
                               # `saage remote` syncs back; ignored by local runs
shared:                        # optional: initial shared-store values
  question: "..."
  target_accuracy: 0.97
workflow:                      # required: an ordered list of steps
  - <step>
  - <step>
```

- `provider:` is optional. Omit it (the norm) and the flow runs with the user's
  `saage setup` defaults; include it only to pin the flow to a provider+model it
  genuinely needs (a pin beats the defaults, and must carry both `type` and
  `model`). `provider.type` ∈ `anthropic | openai | openrouter | nvidia |
  local`; optional `retry: { max_attempts, base_delay }` sub-block, optional
  `request_timeout` (cap in seconds on a single model call — raise for slow
  `local` servers whose thinking turns outlive the SDK default of 600s; the
  cap is per retry attempt, so a hung server can cost up to
  `retry.max_attempts x request_timeout`). CLI
  `--provider/--model/--base-url/--request-timeout` beats both at run time.
- `workspace`, `venv`, `flow_dir`, and `python` are auto-seeded into the shared
  store, so `{{ workspace }}` / `{{ flow_dir }}` / `{{ venv }}` / `{{ python }}`
  are available in templates. `python` is the interpreter launcher for helper
  scripts (`python3` on POSIX, `python` on Windows — there is no `python3.exe`).

## Step types (exact YAML)

**`agent`** — run an LLM skill with the harness tools.
```yaml
- { id: write_query, type: agent, skill: write_query,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cgpadwick/saage](https://github.com/cgpadwick/saage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
