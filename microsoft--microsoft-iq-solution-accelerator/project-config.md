---
trigger: always_on
description: Use when editing docs/DeploymentGuide.md, the top-level deployment guide for the Microsoft IQ Solution Accelerator. Covers document structure, script and resource references, relative link conventions, and source of truth.
---


# Top-level deployment guide — structure and references

## Document

[`docs/DeploymentGuide.md`](../../docs/DeploymentGuide.md) is the **single end-to-end `azd up` guide** for the Microsoft IQ Solution Accelerator. It must stay short and self-contained. Operational Fabric-only details (manual portal-only flow) live in [`docs/fabric/DeploymentGuideFabricManual.md`](../../docs/fabric/DeploymentGuideFabricManual.md); the post-deployment Copilot Studio (Work IQ) configuration lives in [`docs/copilot/DeploymentGuide.md`](../../docs/copilot/DeploymentGuide.md). Keep the section structure below stable; only edit content within sections unless the deployment flow itself changes.

### Section structure

1. **Introduction** — one-paragraph summary of Fabric IQ + Microsoft Foundry components, followed by a **Table of Contents** sub-section (`### Table of Contents`) that links to every other top-level section (§2 *Deployment Environment Setup* through §8 *Additional Resources*) plus the relevant sub-sections of §5 (*Infrastructure Provisioned*, *Deployment Phases*) and §6 (*Azure Resources*, *Fabric IQ Components*, *Microsoft Foundry Components*, *Environment Variables*, *Next Steps*). Use GitHub-flavored markdown anchors (lowercase, hyphenated, no punctuation). The TOC must be kept in sync with the section headings whenever a section is added, removed, or renamed.
2. **Deployment Environment Setup** — four collapsible (`<details><summary>`) options that prepare the same prerequisites and converge on the common Deployment Commands. Order and wording must stay stable; only edit content within a `<details>` block:
   - *Option 1 · Local Deployment* — install `azd`, Python 3.9+, PowerShell 7+; clone the repo. Auth happens in Deployment Commands, not in this option.
   - *Option 2 · GitHub Codespaces* — launch via the **[Open in GitHub Codespaces badge](https://codespaces.new/microsoft/microsoft-iq-solution-accelerator)** at the top of the option (badge image: `https://github.com/codespaces/badge.svg`) **or** `Code → Codespaces`. The dev container's `postCreateCommand` provisions tooling automatically. Mention `azd auth login --use-device-code` as a fallback for the browser-redirect issue.
   - *Option 3 · Dev Container (VS Code + Docker Desktop)* — same image as Codespaces, run locally; `~/.azure` is bind-mounted from the host so a previous `azd auth login` carries over.
   - *Option 4 · GitHub Actions* — [`./.github/workflows/azure-dev.yml`](../../.github/workflows/azure-dev.yml) runs `azd up --no-prompt` via OIDC. Requires `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID` secrets in the `miq-build` GitHub environment. This option **skips** the Deployment Commands section.
   Each option must link to [Deployment Commands](#deployment-commands) (except Option 4) and reference [`.devcontainer/README.md`](../../.devcontainer/README.md) for Options 2 & 3.
3. **Deployment Commands** — common path for Options 1–3. *Prerequisites* note ("complete one of the setup options above"); single bash code block in this order: `azd auth login` (commented "run only if not already logged in") → optional `azd env set GITHUB_TOKEN` → commented `azd env set AZURE_AI_DEPLOYMENTS_LOCATION` example noting the parameter is required and otherwise prompted → optional commented `azd env set` example overrides for defaulted parameters (e.g. `FABRIC_CAPACITY_SKU_NAME`) → `azd up` → optional `azd env get-values`. Followed by a blockquote linking to [Optional Configuration Variables](#optional-configuration-variables) for fine-grained tuning. Then a *Re-running Deployment* subsection covering idempotency.
4. **Post-Deployment Steps — Work IQ** — thin pointer section. One-paragraph framing that `azd up` covers Fabric IQ and Microsoft Foundry, and that Work IQ (Copilot Studio email-triggered agent) is the **manual** third component. Prominent link to [`docs/copilot/DeploymentGuide.md`](../../docs/copilot/DeploymentGuide.md) as the source of truth. A numbered 4-item summary mirroring the Copilot guide steps (Import solution → Configure connections → Configure email trigger → Publish agent + Teams channel). Connection list must mention the Foundry Agent endpoint comes from `AZURE_AI_AGENT_ENDPOINT` (`azd env get-values`). Mention the Power Platform solution lives at [`src/copilot/sln/MicrosoftIQAccelerator`](../../src/copilot/sln/MicrosoftIQAccelerator). End with a link to [`docs/copilot/README.md`](../../docs/copilot/README.md) for architecture overview and [`docs/copilot/TestingGuide.md`](../../docs/copilot/TestingGuide.md) for QA. **Do not duplicate** connection-by-connection details from the Copilot guide — keep this section as a pointer.
5. **Optional Configuration Variables** — single table of every `azd env set` variable that affects deployment behavior, plus the four "Available" lists (Fabric SKUs, AI deployment regions, use cases, deployment types). Must stay in sync with [`infra/main.parameters.json`](../../infra/main.parameters.json) and [`azure.yaml`](../../azure.yaml).
6. **Deployment Overview**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/microsoft-iq-solution-accelerator](https://github.com/microsoft/microsoft-iq-solution-accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
