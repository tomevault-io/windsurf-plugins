---
trigger: always_on
description: Runbooks for a coding agent that runs, operates or verifies this repository's training on
---

# Agents that operate training

Runbooks for a coding agent that runs, operates or verifies this repository's training on
its user's behalf. The human documents say how the recipe works; this file says what an
agent must do and must not do with the scripts. It describes only what the code and those
documents show. Decisions that cost money (a host, a reservation) belong to the user.
Paths are relative to the repository root. Replace every `<...>` placeholder.

| runbook | when |
| --- | --- |
| [0. Ground rules](#0-ground-rules) | always |
| [1. Get a GPU host](#1-get-a-gpu-host) | before any GPU work |
| [2. Set up and verify a host](#2-set-up-and-verify-a-host) | after every launch, and after every code change |
| [3. Run the recipe](#3-run-the-recipe) | a retrain, or one stage of it |
| [4. Verify a code change before you trust a retrain](#4-verify-a-code-change-before-you-trust-a-retrain) | the code changed since the published checkpoint |
| [5. Evaluate on JevBench](#5-evaluate-on-jevbench) | a checkpoint to score |
| [6. Export and publish a checkpoint](#6-export-and-publish-a-checkpoint) | a retrain to keep |
| [7. Tear down](#7-tear-down) | the end of every session |

Human documents: [training/README.md](README.md) (the recipe, the reproduction bar),
[training/aws/README.md](aws/README.md) (the scripts, every variable, timings, costs),
[training/steps.md](steps.md) (the six steps), [evaluation/jevbench.md](../evaluation/jevbench.md)
(the benchmark), [data/README.md](../data/README.md) (what is committed, what is downloaded).

## 0. Ground rules

- **Spend is the user's decision.** Launching a host, buying a Capacity Block and creating
  a reservation all cost money. Ask your user before each one, with the shape, region and
  duration you intend to use. Do not retry a launch or buy anything without that answer.
- **No secrets on a host, in an SSM command or in a log.** Never `echo`, `cat` or paste a
  token or key. `huggingface_hub` reads its token by itself on your machine. The host
  needs no Hugging Face token (the models are public) and no AWS key (its IAM role).
- **One `HOBSON_NAME_TAG` and one `HOBSON_STATE_DIR` per agent.** The scripts find hosts by
  the `Name` tag and `teardown.sh` terminates every host with it. Two agents on one tag
  find and terminate each other's hosts. Set both before the first script call:

  ```bash
  # training/aws/scripts/local.env is git-ignored and sourced first. Every line needs `export`:
  # common.sh does not export what it sources, so a plain AWS_PROFILE=... never reaches the CLI.
  export AWS_PROFILE=<profile>
  export HOBSON_BUCKET=<bucket>                         # shared by every agent of the account
  export HOBSON_NAME_TAG=hobson-<agent>                 # unique per agent
  export HOBSON_JOB_TAG=<job>
  export HOBSON_STATE_DIR=training/aws/scripts/.state/<agent>
  export HOBSON_REGIONS="<every region you may launch in>"
  ```

- **Tear down every host you launch and confirm `terminated`.** Cancel every reservation
  you create. Report instance ids, launch and termination times, and reservation ids to
  your user.
- **Long jobs run in `hobson-bg`, never in a held-open SSM call.** `ssm-run.sh` has a
  one-hour default timeout, and a finished call is the only way to get its output.
- **Poll, do not spin.** One `ssm-run.sh` status call every few minutes; the AWS API
  throttles tight loops.
- **Ship commits, not working trees.** `sync-code.sh` archives one commit. Uncommitted
  changes never reach the host, and the host records the commit as `git_rev`.
- **Record every deviation** from a documented command: what, when, why, what you changed.
- **Never modify the benchmark**, and never change a script to make a check pass.

## 1. Get a GPU host

**When.** Before any stage, evaluation or gate that needs a GPU. How many GPUs each route
needs, and how long it takes, is in [training/README.md](README.md) and
[Measured stage timings](aws/README.md#measured-stage-timings). Read
[What you need](aws/README.md#what-you-need) once: credentials, default VPC, vCPU quota.
Ask your user before you launch.

**Commands.** `launch-host.sh` launches one On-Demand host ([2. A host](aws/README.md#2-a-host)).
It tries every shape in `SHAPES` in every AZ of every region in `REGIONS`, moves on at
each capacity error, and stops at any other error. It is idempotent: if a host with your
tag exists, it prints that host and exits.

```bash
training/aws/scripts/launch-host.sh                                  # defaults: see the variable table in aws/README.md
SHAPES="<shape ...>" REGIONS="<region ...>" ROOT_GB=<GiB> training/aws/scripts/launch-host.sh
```

Every region in `REGIONS` must be in `HOBSON_REGIONS` too, or `teardown.sh` does not find
the host. `ROOT_GB` is the size of the gp3 root volume (default 1024 GiB); the host's
data, checkpoints and model cache live on the NVMe scratch, not on it. Every attempt is
logged in `$HOBSON_STATE_DIR/launch-attempts.log`.

`launch-capacity-block.sh` launches one host into an EC2 Capacity Block that already
exists. It waits until the reservation is `active`, launches into it, and sets the safety
poweroff 30 min before the block ends (`SHUTDOWN_AT` overrides). It does not buy the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-labs/strands-decider](https://github.com/strands-labs/strands-decider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
