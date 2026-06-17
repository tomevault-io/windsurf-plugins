---
trigger: always_on
description: This repository is an **AI Landing Zone implemented in Bicep** for Azure.
---

## Solution Overview

This repository is an **AI Landing Zone implemented in Bicep** for Azure.
It provides a reusable, production-oriented infrastructure baseline for AI workloads built on Microsoft Foundry and related Azure services.

Primary goals:
- Standardize secure, repeatable provisioning with `azd` + Bicep.
- Offer modular deployment with feature flags and strong parameterization.
- Support both quick-start and hardened network-isolated topologies.
- Act as a reusable infra core for other accelerators and agent-based solutions.

Repository:
- https://github.com/azure/bicep-ptn-aiml-landing-zone

---

## What To Understand First

This is an IaC-first repository. Most work happens in:
- `main.bicep`: orchestrates all resources, modules, conditions, identities, role assignments.
- `main.parameters.json`: deployment topology, feature flags, model list, app list, networking options.
- `modules/`: reusable custom Bicep modules for networking, security role assignment, AI Foundry connections, app config population.
- `constants/constants.bicep`: role IDs and naming abbreviations.
- `azure.yaml`: azd project definition (`infra.path: .`, `infra.module: main`).
- `manifest.json`: release metadata + optional component repos + jumpbox install script source.
- `install.ps1`: jumpbox bootstrap logic for isolated environments.

If behavior changes, update Bicep and parameters consistently.

---

## IaC Architecture and Design Patterns

### 1) Single entrypoint orchestration
- `main.bicep` is the deployment orchestrator.
- It composes AVM modules and custom modules using feature-flag conditions (`if (...)`).

### 2) Feature-flag driven composition
Common toggles:
- `deployAiFoundry`
- `deployAppConfig`
- `deployKeyVault`
- `deploySearchService`
- `deployStorageAccount`
- `deployCosmosDb`
- `deployContainerEnv`
- `deployContainerApps`
- `networkIsolation`
- `useExistingVNet`

Preserve this conditional model when adding resources.

### 3) Strong parameterization + substitution model
- `main.parameters.json` supports env var substitution (`"${ENV_NAME}"`) for `azd` workflows.
- In Bicep, values that can come empty should have safe fallback handling.
- Avoid hardcoding tenant/subscription/resource names in templates.

### 4) Modular networking for Zero Trust
- Supports public and isolated modes.
- Isolated mode includes VNet/subnets, private DNS zones, private endpoints, and controlled dependencies.
- PE creation is serialized in places to avoid parallel conflicts.

### 5) Identity and RBAC by design
- Supports system-assigned MI and optional UAI (`useUAI`).
- Role assignments are explicit and centralized, including data-plane Cosmos assignments.

### 6) App topology as data
- `containerAppsList`, `modelDeploymentList`, `databaseContainersList`, `storageAccountContainersList` drive infra shape.
- New app/model/service behavior should be added by extending these lists and mapping logic.

---

## Current Container App Port Behavior

Container app ingress and Dapr app port are parameterized per app entry:
- `app.target_port` when provided.
- fallback to `8080` when omitted.

Pattern in `main.bicep`:
- `ingressTargetPort: int(app.?target_port ?? 8080)`
- `dapr.appPort: int(app.?target_port ?? 8080)`

Implications:
- If you add apps in `main.parameters.json`, you can set `target_port` explicitly.
- If omitted, app config defaults to 8080.

---

## Parameterization Guidance

When adding new capability, follow this sequence:
1. Add parameter in `main.bicep` with description and sensible default.
2. Add corresponding value in `main.parameters.json` (literal or `"${ENV_VAR}"`).
3. If substitution can resolve to empty, add fallback handling in Bicep.
4. Wire the value to modules/resources.
5. If runtime needs the value, publish it to App Configuration through `appConfigPopulate`.
6. If downstream automation needs it, expose as Bicep output.

Good practices:
- Keep booleans as true/false semantics in Bicep.
- Keep names deterministic (resource token + abbreviations).
- Keep module params minimal but explicit.
- Preserve idempotency.

---

## Reusing This Landing Zone from Another Repository

This section is critical for derived accelerators.

### Recommended consumption model
Use this repository as an **infra submodule** mounted at `infra/` in the consumer repo.

Why:
- Consumer gets a stable IaC core.
- Consumer customizes only overlays (`main.parameters.json`, `manifest.json`, optional scripts).
- Infra updates are versioned by submodule pin.

### Pinning strategy for consistency
Pin submodule to a specific landing zone release/tag.

Example commands to add and pin the submodule to `v1.0.0`:
```bash
git submodule add https://github.com/Azure/bicep-ptn-aiml-landing-zone.git infra
git -C infra fetch --tags
git -C infra checkout tags/v1.0.0
git config -f .gitmodules submodule.infra.branch v1.0.0
git config -f .gitmodules submodule.infra.ignore dirty
git add .gitmodules infra
git commit -m "Add infra submodule pinned to v1.0.0"
```

Initialization command for consumers:
```bash
git submodule update --init --recursive
```

Example `.gitmodules` pattern:
```ini
[submodule "infra"]
	path = infra
	url = https://github.com/Azure/bicep-ptn-aiml-landing-zone.git
	branch = v1.0.0
	ignore = dirty
```

Notes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/bicep-ptn-aiml-landing-zone](https://github.com/Azure/bicep-ptn-aiml-landing-zone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
