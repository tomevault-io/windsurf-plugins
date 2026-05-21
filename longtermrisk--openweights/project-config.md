---
trigger: always_on
description: OpenWeights is a Python SDK for running distributed compute jobs on managed RunPod GPU infrastructure. It provides a simple, OpenAI-like API with full flexibility for custom workloads including fine-tuning, inference, evaluations, and arbitrary Python scripts.
---

# OpenWeights Architecture

## Overview

OpenWeights is a Python SDK for running distributed compute jobs on managed RunPod GPU infrastructure. It provides a simple, OpenAI-like API with full flexibility for custom workloads including fine-tuning, inference, evaluations, and arbitrary Python scripts.

**Key Features:**
- Simple Python SDK with OpenAI-compatible interfaces
- Full flexibility to define custom jobs with arbitrary Docker images and entrypoints
- Automated management of RunPod GPU infrastructure
- Multi-tenancy with organization-based isolation
- Content-addressable job and file IDs for deduplication

## Core Concepts

### What is a Job?

A job is the fundamental unit of work in OpenWeights. It consists of three components:

1. **Docker Image**: The container environment (e.g., `nielsrolf/ow-unsloth`, `nielsrolf/ow-vllm`, or custom images)
2. **Mounted Files**: Files uploaded to Supabase storage and mounted into the container
3. **Entrypoint**: The command/script to execute (e.g., `python train.py --model=llama`)

Jobs can be:
- **Built-in jobs**: Pre-configured templates for common tasks (fine-tuning with Unsloth, inference with vLLM, Inspect AI evaluations)
- **Custom jobs**: User-defined jobs using the `@register` decorator and `Jobs` base class

### Job Lifecycle States

Jobs progress through the following states:
- `pending`: Job is queued, waiting for a worker
- `in_progress`: Job is currently executing on a worker
- `completed`: Job finished successfully
- `failed`: Job encountered an error
- `canceled`: Job was manually canceled or timed out

### Jobs, Runs, and Events

**Jobs** are reusable templates that define what to execute:
- Identified by content hash of their parameters (e.g., `unsloth-abc123def456`)
- If you submit the same job twice, it uses the existing job (deduplication)
- Contain: docker image, script/entrypoint, parameters, VRAM requirements, hardware constraints

**Runs** are individual executions of a job:
- Each job can have multiple runs (e.g., if restarted after failure)
- Track execution status, assigned worker, and log file
- Created when a worker picks up a job or when using `ow.run` context

**Events** are structured logs/outputs during a run:
- Store arbitrary JSON data (metrics, checkpoints, errors)
- Can reference uploaded files (model checkpoints, outputs)
- Used to track progress and collect results

**Relationship:**
```
Job (1) ──< (many) Runs (1) ──< (many) Events
```

## System Architecture

OpenWeights follows a queue-based architecture with three main components:

### 1. Job Queue (Supabase)

**Database Tables:**
- `jobs`: Job definitions and status
- `runs`: Execution records linking jobs to workers
- `events`: Structured logs and outputs from runs
- `files`: File metadata (actual files stored in Supabase Storage)
- `worker`: Worker registration and health tracking
- `organizations`: Multi-tenant isolation
- `organization_secrets`: API keys and credentials (HF_TOKEN, RUNPOD_API_KEY, etc.)
- `service_account_tokens`: JWT tokens for API authentication

**Key Features:**
- Row Level Security (RLS) ensures organization isolation
- Atomic job acquisition using PostgreSQL functions (`acquire_job`, `update_job_status_if_in_progress`)
- Content-addressable IDs prevent duplicate jobs and files

### 2. Cluster Manager

**Architecture:**
- **Supervisor** (`cluster/supervisor.py`): Top-level process that spawns one manager per organization
- **Organization Manager** (`cluster/org_manager.py`): Manages GPU workers for a single organization

**Responsibilities:**
1. Monitor job queue for pending jobs
2. Provision RunPod workers when jobs arrive
3. Scale workers based on demand (up to MAX_WORKERS per org)
4. Terminate idle workers (idle > 5 minutes)
5. Clean up unresponsive workers (no ping > 2 minutes)
6. Match jobs to hardware based on VRAM requirements and `allowed_hardware` constraints

**Worker Provisioning:**
- Determines GPU type based on job's `requires_vram_gb` and `allowed_hardware`
- Supports multi-GPU configurations (1x, 2x, 4x, 8x GPUs)
- Creates worker record in database with `status='starting'`
- Launches RunPod pod with appropriate Docker image and environment variables
- Updates worker record with `pod_id` when pod is ready

### 3. Workers

**Worker Lifecycle:**
1. **Initialization** (`worker/main.py`):
   - Detects GPU configuration (type, count, VRAM)
   - Runs GPU health checks
   - Registers in database with hardware specs
   - Starts health check background thread

2. **Job Acquisition:**
   - Polls database for pending jobs matching its Docker image
   - Filters by hardware compatibility (VRAM or `allowed_hardware`)
   - Prefers jobs with cached models
   - Uses `acquire_job()` RPC for atomic job claiming

3. **Job Execution:**
   - Downloads mounted files from Supabase Storage
   - Creates temporary directory for job execution
   - Runs job script with `OPENWEIGHTS_RUN_ID` environment variable
   - Streams logs to local file and stdout
   - Monitors for cancellation signals

4. **Result Collection:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longtermrisk/openweights](https://github.com/longtermrisk/openweights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
