---
trigger: always_on
description: HHTools exposes the same versioned H2R, scene-free R2R, and scalable Batch Agent contracts
---

# Agent interfaces

HHTools exposes the same versioned H2R, scene-free R2R, and scalable Batch Agent contracts
through two local adapters. Use the JSON CLI from scripts and use MCP when a compatible coding
agent should discover and call tools directly.

| Adapter | Runtime | Entry point |
|---|---|---|
| JSON CLI | Client of a running WebUI Agent API | `uv run hhtools agent ...` |
| MCP | Own local stdio process; no WebUI required | `uv run --extra mcp hhtools-mcp` |

The current Agent surface supports human-to-robot (H2R) retargeting—plain motion through Newton
and safely inspectable object-interaction or terrain-scene bundles through Interaction-Mesh—and
scene-free robot-to-robot (R2R) trajectories through Newton, plus ordered H2R/R2R batches. It
includes content-bound H2R and R2R pair-calibration status, constrained proposals, deterministic
validation, front/side visual previews, validated silent save, capability and robot
discovery, allowlisted asset registration and inspection, workflow-specific preflight, jobs,
revision-aware waiting, verified artifacts, and export. Scene-bearing R2R, Video2Motion, Analysis,
remote service access, and robot deployment are not part of this interface. Code-capable
source formats still require safe content inspection; the Agent never bypasses an
isolated-validation requirement.

## Install

Use any compatible Python 3.12 or newer and install the adapter you need:

```bash
# JSON CLI plus the resident WebUI service
uv sync --locked --extra web --extra retarget

# Self-contained local MCP H2R/R2R server
uv sync --locked --extra mcp
```

After installation, `uv run hhtools doctor --require mcp` provides a
side-effect-free readiness check. Add `--json` for a single machine-readable
document; optional body-model and GVHMR checks do not fail the default command.

The JSON CLI always emits one strict JSON document. Start the WebUI service,
then query it from another terminal:

```bash
uv run hhtools web
uv run hhtools agent capabilities
uv run hhtools agent --help
uv run hhtools agent preflight batch --request batch-request.json
uv run hhtools agent calibration status --request calibration-status.json
uv run hhtools agent calibration propose --request calibration-proposal.json
uv run hhtools agent calibration validate --request calibration-validation.json
uv run hhtools agent calibration save --request calibration-save.json
uv run hhtools agent calibration r2r status --request r2r-calibration-status.json
uv run hhtools agent calibration r2r propose --request r2r-calibration-proposal.json
uv run hhtools agent calibration r2r validate --request r2r-calibration-validation.json
uv run hhtools agent calibration r2r save --request r2r-calibration-save.json
uv run hhtools agent job wait JOB_ID --after-revision REVISION --wait-timeout 20
```

Use `hhtools agent asset catalog` to discover registerable Motion Library and
Robot Library entries before the registry contains any assets. MCP clients use
the equivalent read-only `list_available_assets` tool. Both return only an
allowlisted `root_id` and portable `relative_path`, never a host path.

`hhtools-mcp` is a stdio server, so start it through an MCP client rather than
an interactive terminal. Its available options can be inspected with
`uv run --extra mcp hhtools-mcp --help`. Batch caps default to `0` (unlimited); server
administrators may opt into positive `--max-batch-items` and
`--max-batch-total-frames` values. Web, desktop-sidecar, and MCP startup all load the same
persisted setting fields unless an explicit CLI or environment value overrides them.

## Safe H2R, R2R, and Batch workflows

For a new run, discover capabilities and inspect every input. H2R binds one motion and one robot;
R2R binds one scene-free robot trajectory, its declared source robot, the target robot, and their
pair calibration. Call the matching preflight with `run_mode: smoke`, then submit only an immutable
plan returned with `status: ready` through `start_job`; retain its `plan_id` and caller-owned key.
Wait by revision and review the evaluation and manifest before considering a
full run. Validated H2R and R2R pair calibrations may be saved automatically; final motion quality
and full-run approval remain human decisions.

H2R calibration assistance is available before preflight or after a
`CALIBRATION_REQUIRED` response. Bind every request to the exact registered robot bundle and
reference, then call `get_calibration_status`, `propose_calibration`, and
`validate_calibration`. Candidates are immutable and content-addressed; revisions name a parent
candidate plus bounded joint overrides instead of editing stored JSON. `preview_calibration`
returns a deterministic front/side PNG as MCP image content so a vision-capable GPT client can
inspect limb direction, symmetry, trunk attitude, feet, and semantic-target mistakes. Current
[OpenAI models](https://developers.openai.com/api/docs/models) support image input and vision,
but the HHTools server cannot authenticate a client model from a self-reported name;
`model_hint` is audit metadata only.

When deterministic validation is still current, `save_calibration` may write a user-overlay
calibration with `validated_silent`. A GPT client that actually inspected the preview uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roboparty/human-humanoid-tools](https://github.com/Roboparty/human-humanoid-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
