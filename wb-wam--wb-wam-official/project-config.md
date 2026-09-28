---
trigger: always_on
description: These instructions apply throughout this repository. Follow the user's selected task and read the relevant component guide before changing or running that component. A code or documentation edit does not require environment provisioning or robot access.
---

# Working in WB-WAM

These instructions apply throughout this repository. Follow the user's selected task and read the relevant component guide before changing or running that component. A code or documentation edit does not require environment provisioning or robot access.

## Conversation

- Reply in the user's language. Ask only for missing information; reuse answers and authorization already supplied in the session.
- Ask at most three short questions per turn, preferably one or two. Inspect discoverable PC and robot facts yourself; ask only for missing information or physical actions that require the user.
- When an answer is required, ask in the final response and end the turn. Resume on the user's next message. Do not keep the turn active with sleep loops, status polling, or repeated reminders. Follow any host-required question-tool contract without adding an idle waiting loop.
- Complete short, independent checks before asking. Ending a turn for user input does not mean the overall task is complete. Silence is not an answer or authorization.

## Repository map

| Area | Purpose / reference |
| --- | --- |
| `training/` | WB-WAM training and policy code; [training guide](training/README.md) |
| `bridge/` | Real-robot policy deployment; [deployment guide](bridge/README.md) |
| `collector/sonic/` | SONIC/PICO collection, device probes and robot-side services; [collection guide](collector/sonic/README.md) |
| `collector/humanoid_gpt/` | Humanoid-GPT collection; [collection guide](collector/humanoid_gpt/README.md) |
| `benchmark/humanoidarena/` | Simulation evaluation; [evaluation guide](benchmark/humanoidarena/README.md) |
| `tracker/` | SONIC and Humanoid-GPT controllers |
| `scripts/env/` | Scoped environment helpers; [environment guide](scripts/env/README.md) |
| `third_party/` | Vendored dependencies and submodules; avoid unrelated changes |

## Environment setup requests

Read and follow [docs/agent_setup.md](docs/agent_setup.md) before configuring training, deployment, collection or HumanoidArena evaluation. It contains the detailed commands, configuration templates, staged questions and completion criteria. Select only the requested workflow.

- **The agent may execute real-robot deployment and collection on the PC and robot.** This includes SSH/SCP, dependency installation, approved downloads, source copying/building, local configuration, hardware checks and service startup/shutdown. Complete executable steps directly instead of requiring the user to run commands. An environment request authorizes ordinary reversible setup; live hand/body control and policy execution additionally require the intended motion to be authorized and the on-site operator ready. Reuse existing authorization. Keep processes observable, honor stop requests and verify cleanup. Repository code/documentation edits alone do not require robot access.
- **Training / deployment:** ask whether to download pretrain, midtrain, both, or neither/reuse existing checkpoints. Confirm the selected downloads before starting them, then execute the approved downloads.
- **HumanoidArena evaluation:** ask about the evaluation task and its task-specific checkpoint. Do not offer or download pretrain/midtrain or training datasets for evaluation alone.
- **Collection:** real-robot deployment setup must pass before collection setup. For a direct collection request, first complete and verify the shared robot/PC deployment in section 4B of `docs/agent_setup.md`, then continue to section 4C in the same conversation. Reuse verified results; failed or unverified deployment checks block collection-specific installation/configuration and PICO/MANUS probes. Follow the motion authorization and readiness requirements below. For collection alone, WB-WAM policy installation/weights are not required; offline conversion and simulation-only requests do not require robot deployment.
- For Chinese requests, default Hugging Face downloads to `https://hf-mirror.com` with proxy variables cleared for the download process only. For other languages, ask for the preferred endpoint. Do not silently switch endpoints or enable a proxy after failure.
- Create and populate the applicable local configuration from its example on each host, including `training/.env`, deployment and collection files. Preserve existing unrelated values and check effective configuration through the actual loader. Unchanged examples and placeholders do not constitute completed setup.
- Keep environments separate: training and the Arena policy server use `wbwam`; the Arena simulator uses its own environment; deployment uses `bridge/.venv-wam`; SONIC/PICO collection uses `.venv_teleop`; robot services use a robot-side environment. Read the component guide for other workflows. The root `pyproject.toml` configures tooling and is not an installable root package.

## Real hardware


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WB-WaM/WB-WAM-Official](https://github.com/WB-WaM/WB-WAM-Official) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
