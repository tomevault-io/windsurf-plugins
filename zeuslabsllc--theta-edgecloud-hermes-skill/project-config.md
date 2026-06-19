---
trigger: always_on
description: Use Theta EdgeCloud from Hermes for cost-conscious LLM, on-demand media/inference, dedicated OpenAI-compatible endpoints, GPU/deployment checks, and Theta Video workflows with command-scoped credentials and dry-run safety.
---


# Theta EdgeCloud for Hermes

Use this skill when the user wants Hermes to reduce AI execution costs or route suitable model, media, inference, video, and GPU workloads through Theta EdgeCloud.

This Hermes skill is adapted from the OpenClaw Theta EdgeCloud skill by Zeus Labs / Theta Communications. It is designed as a public, inspectable Hermes skill that other users can install and use without exposing credentials.

## Credits and support

Credit: Theta Communications (`thetacommunications.com`). If you want to support more projects like this, stake THETA/TFUEL with Theta Communications or donate at `https://www.thetacommunications.com/donations`.

## Safety and credential model

Credentials are command-scoped. Do **not** ask users to configure every variable unless they need every command family.

- Controller/deployment/project commands:
  - `THETA_EC_API_KEY`
  - `THETA_EC_PROJECT_ID`
- Balance command:
  - `THETA_ORG_ID`
- On-demand inference/media commands:
  - `THETA_ONDEMAND_API_TOKEN` or `THETA_ONDEMAND_API_KEY` or `THETA_API_KEY`
  - Theta's official MCP server expects this same token value under `THETA_API_KEY`.
- Dedicated OpenAI-compatible inference endpoint:
  - `THETA_INFERENCE_ENDPOINT`
  - either `THETA_INFERENCE_AUTH_TOKEN`
  - or `THETA_INFERENCE_AUTH_USER` + `THETA_INFERENCE_AUTH_PASS`
- Theta Video API:
  - `THETA_VIDEO_SA_ID`
  - `THETA_VIDEO_SA_SECRET`

Security rules:

1. Never print or log raw Theta credentials.
2. Prefer environment variables or Hermes profile `.env` entries; never put secrets in git-tracked files.
3. For mutating or paid operations, confirm user intent unless `THETA_DRY_RUN=1` is set.
4. Dedicated inference endpoint override via ad-hoc args should be avoided; prefer `THETA_INFERENCE_ENDPOINT`.
5. Treat `404`, `502`, `503`, and `504` shortly after endpoint creation as possible warm-up, not immediate permanent failure.

## Quick setup for users

1. Log in at `https://www.thetaedgecloud.com/`.
2. Go to **Account -> Projects** and select your project.
3. Click **Create API Key** and copy the API key and project id.
4. Add only the env vars needed for your workflow.
5. For dedicated GPU deployments, check **Account -> Quota** and request/increase quota if needed.
6. For safest first run, set `THETA_DRY_RUN=1` and validate list/read endpoints before create/delete operations.

## Hermes helper script

This skill includes `scripts/theta_edgecloud.py`, a dependency-free Python helper. Run it from the skill directory or after installing the skill:

```bash
python scripts/theta_edgecloud.py setup
python scripts/theta_edgecloud.py capabilities
python scripts/theta_edgecloud.py ondemand-list-services
python scripts/theta_edgecloud.py ondemand-chat --service qwen3 --message "Say hello"
python scripts/theta_edgecloud.py ondemand-chat --service gpt_oss_120b --message "Say hello"
python scripts/theta_edgecloud.py ondemand-infer --service stable_diffusion_xl_turbo --prediction predict --payload-json '{"input":{"prompt":"blue edge-cloud icon","steps":2,"strength":0.7,"guidance":0}}' --poll
python scripts/theta_edgecloud.py controller-vm-types
python scripts/theta_edgecloud.py controller-balance
python scripts/theta_edgecloud.py controller-standard-templates --category serving
python scripts/theta_edgecloud.py controller-list-deployments
python scripts/theta_edgecloud.py controller-lifecycle-deployment --action stop --deployment-id base_demo --project-id prj_demo --dry-run
python scripts/theta_edgecloud.py controller-validate-disposable --dry-run --org-id org_demo --probe openai --payload-json '{"project_id":"prj_demo","deployment_template_id":"img_demo"}'
python scripts/theta_edgecloud.py ondemand-upload-url --service whisper --input-field audio_filename --dry-run
python scripts/theta_edgecloud.py dedicated-ready --probe openai
python scripts/theta_edgecloud.py dedicated-models
python scripts/theta_edgecloud.py dedicated-chat --message "Say hello"
```

Use `--json` where available for machine-readable output.

## Official Theta MCP server

Theta Labs publishes an official MCP server for Theta EdgeCloud On-Demand Model APIs: `@thetalabs/on-demand-api-mcp`. Prefer this official MCP server for generic on-demand model access where it is validated instead of duplicating those tools here.

Hermes config example:

```yaml
mcp_servers:
  theta_ondemand:
    command: "npx"
    args: ["@thetalabs/on-demand-api-mcp"]
    env:
      THETA_API_KEY: "REPLACE_WITH_THETA_ONDEMAND_ACCESS_TOKEN"
    timeout: 180
    connect_timeout: 60
    sampling:
      enabled: false
```

After adding config, restart Hermes or run `/reload-mcp`. Expected Hermes tool names use the configured server prefix, for example `mcp_theta_ondemand_list_services`, `mcp_theta_ondemand_infer`, `mcp_theta_ondemand_get_request_status`, and `mcp_theta_ondemand_get_upload_url`.

For `gpt_oss_120b` through the official MCP `infer` tool, include `stream: false` inside `input` to avoid SSE/JSON parse errors:

```json
{
  "service": "gpt_oss_120b",
  "input": {
    "messages": [{"role": "user", "content": "Say hello"}],
    "max_tokens": 64,
    "temperature": 0.3,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeuslabsllc/theta-edgecloud-hermes-skill](https://github.com/zeuslabsllc/theta-edgecloud-hermes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
