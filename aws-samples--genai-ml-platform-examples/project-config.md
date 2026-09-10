---
trigger: always_on
description: Samples for governing ML models with managed MLflow on Amazon SageMaker AI and
---

# AGENT.md

Samples for governing ML models with managed MLflow on Amazon SageMaker AI and
the SageMaker Model Registry sync (`AutoModelRegistrationEnabled`). Three
runnable topologies, each a folder with numbered step scripts:

- `topology-1-single-account/` — governance boundary = IAM role
- `topology-2-hub-spoke-central/` — hub-owned MLflow app + registry, RAM sharing
- `topology-3-hub-spoke-hybrid/` — spoke-local everything; approval-triggered
  copy into the hub as a governance record; deployment stays in the spoke

This file is the validated testing guide (profiles, provisioning, env vars, run
order, expected outputs, pitfalls, teardown) for both humans and agents.

## Working conventions

- Ask the user for the two profile names (spoke + hub) before provisioning;
  never guess. Confirm before creating billable infrastructure (MLflow apps,
  Studio domains, `ml.m5.xlarge` endpoints).
- Run step scripts from their topology directory with the repo venv
  (`../.venv/bin/python scripts/NN_*.py`); steps share state via
  `scripts/.sample_state.json` and must run in order.
- Long steps (training ~5 min, endpoint ~4 min) should run in the background
  with output redirected to a log file, then polled.
- After any deploy step succeeds, run the topology's cleanup step promptly —
  endpoints bill per instance-hour.

## Accounts and profiles

Two AWS CLI profiles are required (any names; the scripts read them from env):

- **Spoke / development** — where data scientists work. Topology 1 needs only this.
- **Hub / governance** — owns the central registry. Needed for topologies 2 and 3.

Verify both resolve before doing anything:

```bash
aws sts get-caller-identity --profile <spoke-profile>
aws sts get-caller-identity --profile <hub-profile>
```

Region: validated in `us-west-2`. Managed MLflow apps + Model Registry sync are
not available in every region.

## Provision (CloudFormation, per account)

Each participating account needs one stack from `cfn/sagemaker-studio-mlflow.yaml`
(Studio domain, execution role, S3 artifact bucket `sagemaker-<region>-<acct>`,
managed MLflow app with `AutoModelRegistrationEnabled`). No required parameters.

Workflow:

1. Verify each profile resolves with `aws sts get-caller-identity --profile <p>`
   and report the account IDs.
2. Check whether a `mlflow-governance` stack already exists in each account; if
   so, skip creation and just read the outputs.
3. Confirm with the user before creating anything (managed MLflow apps and
   Studio domains bill while they exist), then deploy per account. Prefer
   `create-stack` + polling over a blocking wait — the MLflow app takes
   ~10 minutes per stack.

```bash
# Spoke (needed for all topologies)
aws cloudformation deploy --template-file cfn/sagemaker-studio-mlflow.yaml \
  --stack-name mlflow-governance --capabilities CAPABILITY_IAM \
  --parameter-overrides DomainName=mlops-dev-domain MLflowAppName=mlflow-dev \
  --profile <spoke-profile> --region us-west-2

# Hub (needed for topologies 2 and 3)
aws cloudformation deploy --template-file cfn/sagemaker-studio-mlflow.yaml \
  --stack-name mlflow-governance --capabilities CAPABILITY_IAM \
  --parameter-overrides DomainName=mlops-hub-domain MLflowAppName=mlflow-hub \
  --profile <hub-profile> --region us-west-2
```

- If the bucket `sagemaker-<region>-<acct>` already exists, stack creation
  fails — delete or adapt first; surface this instead of retrying.
- When `CREATE_COMPLETE`, read the outputs you need (`MLflowAppArn`,
  `SageMakerExecutionRoleArn`):

```bash
aws cloudformation describe-stacks --stack-name mlflow-governance \
  --query "Stacks[0].Outputs[?OutputKey=='MLflowAppArn' || OutputKey=='SageMakerExecutionRoleArn'].[OutputKey,OutputValue]" \
  --output table --profile <profile> --region us-west-2
```

## Python environment

Use **Python 3.12** (matches the `py312` scikit-learn container used for training
and serving). All three topologies share the same pinned `requirements.txt`
(`sagemaker>=3,<4`, `mlflow<4`, `sagemaker-mlflow>=0.5`, `scikit-learn 1.4.x`):

```bash
python3.12 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Run every step script **from its topology directory** so relative paths resolve.
Each topology persists progress in `scripts/.sample_state.json` — steps must run
in order, and a stale state file from a previous run should be deleted before
step 1.

## Run order and env vars per topology

### Topology 1 — single account (spoke only)

```bash
export AWS_PROFILE=<spoke-profile> AWS_DEFAULT_REGION=us-west-2
export MLFLOW_APP_ARN=<spoke MLflowAppArn>
export EXECUTION_ROLE=<spoke SageMakerExecutionRoleArn>
cd topology-1-single-account
python scripts/01_train_and_register.py   # ~5 min: real SageMaker Training Job
python scripts/02_govern_lifecycle.py     # staging OK, production explicitDeny, officer approves
python scripts/03_deploy_and_invoke.py    # ~4 min: real ml.m5.xlarge endpoint
python scripts/04_cleanup.py
```

### Topology 2 — hub-and-spoke central

```bash
export HUB_PROFILE=<hub-profile> SPOKE_PROFILE=<spoke-profile> AWS_DEFAULT_REGION=us-west-2
export HUB_MLFLOW_APP_ARN=<hub MLflowAppArn>
export SPOKE_EXECUTION_ROLE=<spoke SageMakerExecutionRoleArn>
cd topology-2-hub-spoke-central

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/genai-ml-platform-examples](https://github.com/aws-samples/genai-ml-platform-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
