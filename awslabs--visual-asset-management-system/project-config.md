---
trigger: always_on
description: Auto-loaded when Claude Code operates within `infra/lib/nestedStacks/pipelines/`. Covers pipeline stack layout, required Lambda package layout in `backendPipelines/`, VPC builder wiring, sub-process and log registration wiring, and S3 output path conventions. See `infra/CLAUDE.md` for cross-stack patterns (lambda builder, service helper, security helpers).
---

# CLAUDE.md -- VAMS Pipeline Nested Stacks

Auto-loaded when Claude Code operates within `infra/lib/nestedStacks/pipelines/`. Covers pipeline stack layout, required Lambda package layout in `backendPipelines/`, VPC builder wiring, sub-process and log registration wiring, and S3 output path conventions. See `infra/CLAUDE.md` for cross-stack patterns (lambda builder, service helper, security helpers).

---

## Pipeline Nested Stack Pattern

Each pipeline follows a consistent structure:

```
lib/nestedStacks/pipelines/{category}/{pipelineName}/
    {pipelineName}Builder-nestedStack.ts    # Stack definition
    constructs/
        {pipelineName}-construct.ts         # Infrastructure construct
    lambdaBuilder/
        {pipelineName}Functions.ts          # Lambda builder functions
```

**CRITICAL — Pipeline Lambda Directory Structure:** Every pipeline's `lambda/` directory in `backendPipelines/` MUST include:

```
lambda/
  __init__.py                    # Package marker (copy from existing pipeline)
  customLogging/
    __init__.py                  # Package marker
    logger.py                    # safeLogger + mask_sensitive_data (copy from existing pipeline)
  vamsExecute*.py                # Pipeline handler(s)
  constructPipeline.py           # Batch job definition builder
  openPipeline.py                # Step Functions starter
  pipelineEnd.py                 # Cleanup + task token callback
```

Without `__init__.py` and `customLogging/logger.py`, Lambda will fail at import time with `No module named 'customLogging'`. Copy these files from any existing pipeline (e.g., `backendPipelines/3dRecon/splatToolbox/lambda/`).

Pipelines are conditionally created in `pipelineBuilder-nestedStack.ts` based on config flags.

**CRITICAL — VPC Builder Updates:** A new pipeline using AWS Batch, ECS, or Fargate must be added to condition blocks in `lib/nestedStacks/vpc/vpcBuilder-nestedStack.ts` — **which ones depends on the subnets its compute runs in.** Decide that first, by looking at what `pipelineBuilder-nestedStack.ts` passes as the pipeline's `pipelineSubnets`: `pipelineNetwork.isolatedSubnets.pipeline` or `pipelineNetwork.privateSubnets.pipeline`. Search for `useSplatToolbox` (private) and `usePreview3dThumbnail` (isolated) to see both treatments.

| Block                                                                       | Isolated-subnet pipeline | Private-subnet pipeline |
| --------------------------------------------------------------------------- | ------------------------ | ----------------------- |
| 1. **Subnet creation** — pushes `subnetPublicConfig`/`subnetPrivateConfig`  | **No**                   | **Yes**                 |
| 2. **Pipeline-only endpoints** — Batch, ECR API, ECR Docker, optionally EFS | **Yes**                  | **Yes**                 |
| 3. **ECS endpoint** — the `needsEcsPrivate` variable                        | **No**                   | **Yes**                 |

-   **Block 2 is required either way.** Without it, Batch jobs cannot pull their container image, and the pipeline fails at task start with no obvious cause.
-   **Block 1 for a private-subnet pipeline only.** `subnetPrivateConfig` is `PRIVATE_WITH_EGRESS` and the `ec2.Vpc` sets no `natGateways`, so CDK creates **one NAT gateway per Availability Zone** (~$66/month at the default two AZs, plus data processing). Add an isolated-subnet pipeline here and that cost is incurred for subnets its ENIs never occupy. Omit it for a private-subnet pipeline and its compute environment fails with `"Resource subnets are required"`.
-   **Block 3 for a private-subnet pipeline only.** This is the ECS **control-plane** endpoint, which the ECS agent on an EC2-launch-type container instance needs. **Fargate tasks do not use it** — they need ECR, Amazon S3 and CloudWatch Logs, which block 2 supplies. Each endpoint adds one ENI per AZ (~$15/month).

Six pipelines run in isolated subnets today (3dBasic, CAD/mesh metadata extraction, Potree viewer, 3D thumbnail, GenAI metadata labeling, coordinate transform) and appear in block 2 only. Four run in private subnets (Splat Toolbox, NVIDIA Cosmos, NVIDIA GR00T, Isaac Lab training) and appear in all three. Regression coverage: `infra/test/pipelines/coordinateTransformVpcPlacement.test.ts`, which asserts both directions — no NAT for an isolated-subnet pipeline, NAT present for a private-subnet one.

### Sub-Process and Log Registration Wiring

The lambda that starts the pipeline's state machine (or submits a Batch job itself) registers its sub-process and log sources on the orchestration bus (`backendPipelines/CLAUDE.md` "Registering Sub-Processes and Logs"). Its builder supplies:

-   `ORCHESTRATION_BUS_NAME: orchestrationBus.eventBusName` and `orchestrationBus.grantPutEventsTo(fun)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/visual-asset-management-system](https://github.com/awslabs/visual-asset-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
