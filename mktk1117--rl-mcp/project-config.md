---
trigger: always_on
description: rlmcp exists so an agent can *watch, diagnose and steer a live training run*.
---

# Working in this repo (agent guide)

rlmcp exists so an agent can *watch, diagnose and steer a live training run*.
The failure mode this file is written against is real and was observed: an agent
re-implemented half of rlmcp by hand — its own metrics parser, its own frame
tiling, its own stage ladder in a scratch file — because the entry point it
happened to read first did not mention the feature. Read this before deciding
how to drive a run.

This file is the canonical guide. `CLAUDE.md` is a symlink to it, so an
assistant that looks for that name finds the same text; there is one copy to
maintain and nothing here assumes a particular agent product.

**This file is about working *in* this repository.** To *use* rlmcp, read
[docs/tools.md](docs/tools.md) — every command and MCP tool, one entry each —
and [docs/mcp-server.md](docs/mcp-server.md) if you are driving over MCP.
[docs/tuning.md](docs/tuning.md) is the workflow for driving a training run
well: verify the task before training, watch the right numbers, diagnose
before touching a weight.

## This repo is the tool. The records are somewhere else.

Two repositories, on purpose:

* **this one** — the harness, adapters, CLI and MCP server. Its history should
  be about the tool.
* **your records** — your mjlab task packages and `records/`, the run records.
  That repository holds a package per task (env config, `mdp/` terms, its
  `rlmcp` extension, its curriculum, its launcher), the records those runs
  filled, and its own guidance for *running* experiments: simulator traps,
  verification checks, the run-record lifecycle.

rlmcp is a dependency of that repo, never the other way round. Nothing about a
task — no env config, no reward term, no run record — belongs in this tree. If
you are about to add one, you are in the wrong repository.

`$RLMCP_RECORDS` is what separates them at runtime. The records root resolves to
the explicit argument (`rlmcp record --records-root`, `rlmcp.wrap(records_root=...)`),
then `$RLMCP_RECORDS`, then `./records` — see `open_store` in `rlmcp/records/filestore.py`.
A stray `records/` appearing in this checkout means somebody ran a record command
without the variable set; it is a scratch directory, not the records.

## If you are about to hand-roll something, check here first

| you are about to… | use instead |
| --- | --- |
| grep a task package to find out which task ids exist | `rlmcp tasks` — every registered id, which package registered it, and where its runs land |
| write a scratch script that rolls zero actions to see whether a new task works | `rlmcp check --task <id>` — the six gates (including one training iteration), plus what each reward term pays |
| write a loop that builds a task and steps it so you can poke at it | `rlmcp play --policy zero --mode hold` — a built, stepping, steerable session with no viewer |
| parse `metrics.jsonl` / `status.json` yourself | `rlmcp status`, `rlmcp metrics`, `rlmcp plot` |
| write a loop that pokes weights at milestones | a **curriculum** (`StageSchedule`) — see below |
| add a task-specific verb by editing core code | an **extension** (`rlmcp/extensions/`, or your own package) |
| screenshot by rendering + slicing video frames | `rlmcp shot` (add `--where key=value` to pick envs) |
| judge smoothness from reward curves | `rlmcp diagnose` (measures HF power share, jerk, effort) |
| tell the user "it improved" | `rlmcp video` / `rlmcp plot`, then show them |
| script a clip every N iterations of a run | already done — progress clips at 0, 50, 100, 200 … (`rlmcp video --schedule`) |
| start a viewer, or `play`, to see what a run is doing *now* | `rlmcp view` — a run already serves one; this says where. No restart, no renderer |
| explain that the live view "looks too fast" | `rlmcp view --realtime` — a buffered window played back at 1x, with a player in the tab |
| stop the view costing the run anything, without losing the tab | `rlmcp view --pause` — the frame stays, the port stays, the run goes back to full speed |
| look at a policy whose run has already exited | `rlmcp play` — it restores the conditions the checkpoint trained under first |
| call `env.reset()` yourself to clear a bad state | `rlmcp reset-envs` (`--where key=value` to restart only some) |
| restart `rlmcp play` to see a different checkpoint | `rlmcp run load_policy checkpoint=<path>` — same env, same conditions, new weights |
| compare a metric across several runs | `rlmcp record compare`, or `rlmcp record graph` for the ancestry |
| keep the record of an experiment in a scratch file | `rlmcp record new` / `rlmcp record close` — it belongs **in** the records |

## Where things are

```
rlmcp/core/          parameters, telemetry, traces, curriculum, controller — no backend
rlmcp/adapters/      SimAdapter / RunnerAdapter; mjlab/ is the reference implementation
rlmcp/extensions/    capabilities the core does not know about; terrain.py is the model
rlmcp/records/           the records: plans, outcomes, ancestry, the rendered tree
rlmcp/server/        the MCP server (imports no simulator, by design)
```

The layering rule is one sentence: **the core never learns a task's vocabulary,
and the adapter never learns a capability.** Anything a particular simulator has

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mktk1117/rl-mcp](https://github.com/mktk1117/rl-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
