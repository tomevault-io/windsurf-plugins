---
trigger: always_on
description: Keep this repository a small, understandable TPU job runner. The public surface
---

# TPU Runner agent guide

## Project intent

Keep this repository a small, understandable TPU job runner. The public surface
is the `tpu-runner` CLI, a deployment YAML file, and a job YAML file. Prefer
direct code and targeted validation over frameworks, background services, CI
workflows, or large test scaffolds unless the user explicitly asks for them.

The user-facing README should stay short and conceptual. Put detailed
architecture, invariants, race analysis, and maintainer procedures here.

## Module map

- `tpu_runner/specs.py`: immutable fleet/job/cache models and YAML validation.
- `tpu_runner/placement.py`: resolves regional job buckets and rejects literal
  `gs://` dependencies that would defeat local placement.
- `tpu_runner/capacity_policy.py`: pure scheduling, stable priority, compatibility,
  and desired-capacity policy.
- `tpu_runner/runtime.py`: Firestore records, serialization, leases, and all
  multi-record transactional state transitions.
- `tpu_runner/gcp.py`: exact-name TPU and queued-resource descriptions and
  mutations through `gcloud`.
- `tpu_runner/controller.py`: ordered reconciliation of inventory, jobs,
  attempts, cancellations, interruptions, preemption, and capacity.
- `tpu_runner/distributed.py`: all-worker SSH, remote launch/cancel protocol,
  GCS status polling, logs, diagnostics, caches, and process identity.
- `tpu_runner/cli.py`: CLI parsing, submission, observation, deployment entry,
  controller loop, and leader-lease renewal.
- `tpu_runner/deploy.sh`: GCP control-plane provisioning and controller rollout.
- `tpu_runner/startup.sh`: TPU VM user, SSH, scratch, linger, and readiness setup.
- `tpu_runner/Dockerfile` and `cloudbuild.yaml`: controller image build.

## Deployment lifecycle

`tpu-runner deploy deployment.yaml` validates the fleet, then `deploy.sh`:

1. Enables required APIs.
2. Creates the derived `us-central2` runner bucket, or an explicitly named
   runner bucket, if absent.
3. Creates the default Firestore database if absent.
4. Creates controller and worker service accounts.
5. Adds controller roles (`datastore.user`, `compute.viewer`,
   `iam.serviceAccountUser`, `storage.objectViewer`, `tpu.admin`, plus IAP when
   configured).
6. Adds worker roles (`logging.logWriter`, `storage.objectAdmin`) in the runner
   project and grants access to declared pre-existing worker secrets.
7. Creates or reads the runner SSH private key in Secret Manager.
8. Renders and uploads the worker startup script to
   `BUCKET/artifacts/startup.sh`.
9. Uploads the deployment spec to `specs/deployment.yaml`.
10. Stages a temporary package-local build context and builds the controller.
11. Sets a new controller epoch, cancels older executions, waits for the old
    lease to release or expire, deploys the new Cloud Run job, and executes it.

`worker_secrets` contains existing Secret Manager names in the runner project,
never secret values. Deployment grants the worker service account accessor
permission but does not inject values into jobs; workloads retrieve them at
runtime. Job environment values are materialized in Firestore and GCS and must
not contain secrets.

The package-local build context is required for PyPI installations. Do not make
deployment depend on repository-root files that are absent from an installed
wheel. Keep `controller-requirements.txt` aligned with imports needed inside the
controller image.

The packaged `tpu_runner/deployment.example.yaml` is the deployment template.
`tpu-runner init` copies it into the user's working directory.

## Submission lifecycle

`submit_jobs` performs all fallible preparation before publishing jobs:

1. Load the manifest and generate IDs for unnamed jobs.
2. Select requested manifest IDs before external work.
3. Validate TPU name/type/zone compatibility against declared fleet ordinals.
4. Resolve every declared job bucket to one distinct exact region.
5. Reject multi-region jobs with literal GCS references; they must select
   mirrored data through `JOB_BUCKET`.
6. Reproducibly archive local bundles, hash content, and upload to every
   candidate regional bucket only if absent.
7. Upload a uniquely named materialized job spec to every candidate bucket.
8. Atomically create the full set of Firestore job records.
9. Record events and trigger the Cloud Run controller in a `finally` block.

Artifacts may exist without a Firestore job if preparation or atomic creation
fails. That is harmless. A Firestore job must never be visible before its
bundle and materialized spec exist.

## Controller reconciliation order

`Controller.reconcile_once` deliberately runs in this order:

1. Renew the lease and describe exact current/previously known resource names.
2. Refresh queued-resource records and ready TPU records.
3. Recover runner-owned scratch pressure on eligible idle adopted TPUs.
4. Process job cancellations.
5. Process exact controlled-interruption requests.
6. Poll or launch controller-managed attempts.
7. Recycle managed Spot ordinals scheduled after repeated infrastructure or
   cancellation failures.
8. Reconcile missing, terminal, transport-incompatible, or unhealthy resources.
9. Assign pending jobs to already-idle compatible resources.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dhidary/tpu-runner](https://github.com/dhidary/tpu-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
