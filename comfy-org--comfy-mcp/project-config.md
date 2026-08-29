---
trigger: always_on
description: A short guide for AI agents (and humans). **Keep it in sync:** a PR changing the
---

# Agent Guidelines — comfy-mcp

A short guide for AI agents (and humans). **Keep it in sync:** a PR changing the
architecture rule, the toolchain, or the tool set updates it too. `comfy-mcp` is a small,
standalone [MCP](https://modelcontextprotocol.io) server that lets an agent drive a user's
**local** ComfyUI: a **thin wrapper over
[`comfy-cli`](https://github.com/Comfy-Org/comfy-cli)**, the engine.

## The architecture rule — thin wrapper only (read this first)

Every tool is a passthrough to the `comfy` binary — the only way to reach comfy-cli is the
`_run_comfy(*args)` helper in `src/comfy_mcp/server.py`, which shells out to `comfy --json
--where local <args>` (global flags **before** the subcommand), parses comfy-cli's versioned
`envelope/1` result, and returns its `data`. Do not bypass it.

Hard guardrails — a PR breaking any of these should be rejected:

- **Every tool is a `comfy --json --where local` passthrough.** New functionality belongs in
  comfy-cli, exposed here as a thin `_run_comfy` call. A feature that can't be a `comfy`
  subcommand needs a comfy-cli change, not a workaround here.
- **No HTTP client.** This server never talks to ComfyUI (or anything else) over HTTP
  directly — no `httpx`/`requests`/`aiohttp`/`urllib` calls to a server. comfy-cli owns all
  I/O with ComfyUI (reaching a *local* process means shelling out to `comfy`, never opening
  a socket).
- **No code from the cloud MCP.** Do not copy code, patterns, or dependencies from
  `Comfy-Org/comfy-cloud-mcp-server` — a multi-tenant HTTP service with per-session state,
  signed URLs, analytics, and a cloud API client, none of which apply here. This repo is
  local-only, single-process, with no filesystem/multi-tenancy concerns to design around.

A tool may COMPOSE more than one passthrough when the value is in the sequence, not new
logic — `fetch_template` runs `templates fetch` then `validate`, telling the caller whether
the template it wrote can run; `download_model` submits `model download --background` then
polls `model download-status` so a multi-GB transfer doesn't hold the MCP request open. That
stays inside the rule: every call goes through `_run_comfy`, the verdict is comfy-cli's own,
no product behavior is added. Breaching it means deriving the answer here (parsing the
graph, keeping a table of what is "supported") instead of asking the engine — e.g.
`download_model` sizing the file on disk instead of reading its `status`. `install_node`
composes for a different reason: `comfy node install` runs Manager's `cm-cli`, which a
legacy clone under `custom_nodes/` can't provide, so it reads `comfy env`'s manager fields
BEFORE the consent prompt rather than authorize third-party code on a call that can't
succeed. It fails OPEN and shares `workflow_deps`' two helpers so the two can't drift, and
reads its VERDICT from printed text (`_extract_install_failures`) — cm-cli prints a pack's
failure before consulting `--exit-on-fail`, so a pack that never installed came back `ok:
true` — on the standing of `_extract_saved_paths`: no envelope, so text is the only channel
and the verdict is cm-cli's own sentence, matched one-directionally so a wording change
regresses rather than fails every install. `workflow_deps` reads its answer off DISK because
the engine leaves none — `comfy node deps-in-workflow` emits no envelope and REQUIRES an
`--output` path — so the temp-file round trip is the contract, and the manifest goes back as
written bar `failure_log._scrub_text` masking repo-URL credentials. `restart_comfyui`
composes a THIRD pair: `comfy stop --port <p> --dry-run` / `comfy stop --port <p>` show who
holds a leftover port to recycle — never a `psutil`/HTTP check; that verdict stays
comfy-cli's.

The one thing that legitimately lives here rather than in comfy-cli is **MCP protocol
surface** — capabilities comfy-cli can't express. Today that's the per-call confirmation on
tools that can spend money, destroy local state, run third-party code, kill a process, or
expose the machine: `partner_generate`, `run_template`, `run_workflow`,
`switch_comfyui_version`, `install_node`, `update_comfyui` when `target="all"`, the
`launch_comfyui`/`restart_comfyui` pair when `extra_args` would publish ComfyUI to the
network, and again to kill an untracked server. comfy-cli owns the credit-spend interlock
and the durable "always proceed" (`comfy generate consent always`); this server only raises
the confirmation over MCP **elicitation** — the protocol's y/N prompt — then forwards the
answer as `--yes`/`--allow-spend`, or (for the five the CLI doesn't gate at all) refuses to
run the command. It stores no consent of its own — all share one fail-closed body,
`_elicit_approval`; give a new gate its own `_ApprovalWording`, not a second copy. Adding
*product* behavior here is still a guardrail breach; adapting comfy-cli's contract to an MCP
primitive is this repo's job. Project anchoring (`_project_root`, `COMFY_PROJECT`, the
`project` tool) is the same kind of adaptation: comfy-cli resolves its governing `project/1`
by walking up from its own process cwd, assuming a persistent shell session an MCP client's
arbitrary per-call cwd can't provide, so this server passes `cwd=` on its own spawns instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Comfy-Org/comfy-mcp](https://github.com/Comfy-Org/comfy-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
