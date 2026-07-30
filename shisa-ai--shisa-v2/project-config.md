---
trigger: always_on
description: This document explains how to work with the MegaBlocks training environment using Docker containers.
---

# Agent Docker Workflow Documentation

This document explains how to work with the MegaBlocks training environment using Docker containers.

## Container Setup

The training environment uses a ROCm 7.0 PyTorch container that provides all necessary dependencies for MegaBlocks training on MI300X GPUs.

### Starting the Container

Use the provided script to start an interactive container:

```bash
./01-run-docker.sh
```

This script:
- Mounts the current directory (`/root/shisa-v2/train/v2.1/megatron-mi300x`) to `/workspace/shisa-v2.1` inside the container
- Provides GPU access via `/dev/kfd` and `/dev/dri` devices
- Sets up proper permissions and shared memory
- Uses the ROCm 7.0 PyTorch training image optimized for MI300X

## Working with Running Containers

### Finding Active Containers

To see all running containers:

```bash
docker ps
```

Example output:
```
CONTAINER ID   IMAGE                                               COMMAND     CREATED        STATUS        NAMES
abc123456789   rocm/7.0:rocm7.0_pytorch_training_instinct_...     /bin/bash   5 minutes ago  Up 5 minutes  rocm7_container_1727340123
```

### Executing Commands in Running Containers

To execute commands in a running container, use `docker exec`:

```bash
# Interactive shell
docker exec -it <container_name_or_id> /bin/bash

# Single command
docker exec <container_name_or_id> <command>
```

Examples:
```bash
# Get an interactive shell
docker exec -it rocm7_container_1727340123 /bin/bash

# Check GPU status
docker exec rocm7_container_1727340123 rocm-smi

# Run training script
docker exec rocm7_container_1727340123 /bin/bash -c "cd /workspace/shisa-v2.1 && ./gpt2-125m/03-train-dense.sh"
```

### File System Access

- **Host Path**: `/root/shisa-v2/train/v2.1/megatron-mi300x`
- **Container Path**: `/workspace/shisa-v2.1`
- All files are synchronized between host and container
- Scripts, data, and checkpoints are accessible from both environments

## Training Workflow

### 1. Data Preparation

First, generate the training data (run from host or container):

```bash
# Generate SFT dataset for GPT-2 125M (writes to gpt2-125m/data/)
./gpt2-125m/02-generate.sh
```

This creates:
- `./gpt2-125m/data/sft.shisa-v2.1_text_document.bin` (binary data)
- `./gpt2-125m/data/sft.shisa-v2.1_text_document.idx` (index file)
- `./gpt2-125m/data/gpt2-vocab.json` (vocabulary)
- `./gpt2-125m/data/gpt2-merges.txt` (merge rules)

### 2. Standard Training

Run dense GPT-2 125M training:

```bash
# From container
cd /workspace/shisa-v2.1
./gpt2-125m/03-train-dense.sh
```

### 3. MoE Training

Run Mixture of Experts training:

```bash
# From container
cd /workspace/shisa-v2.1

# Default configuration
./gpt2-125m/04-train-moe.sh

# Custom configuration
./gpt2-125m/04-train-moe.sh my_experiment 128 2 2 0.05 16
#                                        ^experiment  ^experts ^capacity ^top_k ^loss_weight ^batch_size
```

### 4. Convert Checkpoints to Hugging Face

After training finishes, convert the Megatron checkpoint into Hugging Face format:

```bash
cd /workspace/shisa-v2.1
./export-hf.sh gpt2-125m/checkpoints/dense_YYYYMMDD_HHMMSS --iteration iter_0002203
```

- Omit `--iteration` to convert the iteration recorded in `latest_checkpointed_iteration.txt`.
- Add `--output /some/path` to control where the Hugging Face files are written (defaults to `<run_dir>_hf`).
- Use `--model-dir <path>` if your tokenizer data lives outside the checkpoint hierarchy.
- Use `--hf-dtype bf16` (default) or another dtype to control the saved Hugging Face weights.

## Training Configuration

Both scripts now use **epoch-based training** with automatic step calculation:

- **EPOCHS=3** (configurable at top of each script)
- **Training steps** calculated automatically from dataset size
- **Save intervals** set to save at the end of each epoch
- **Global batch size**: 512 (optimized for MI300X)

### Calculation Details

Training steps are calculated as:
```bash
TRAINING_STEPS = (NUM_SAMPLES * EPOCHS) / GLOBAL_BATCH_SIZE
```

The number of samples is automatically extracted from the `.idx` file header.

## Monitoring Training

### Logs

Training logs are saved to:
- Standard: `/workspace/shisa-v2.1/gpt2-125m/checkpoints/<run_name>/train.log` (default run names look like `dense_YYYYMMDD_HHMMSS`)
- MoE: `/workspace/shisa-v2.1/gpt2-125m/checkpoints/<experiment_name>/train.log` (defaults to `moe_YYYYMMDD_HHMMSS` when no name is provided)

### Checkpoints

Checkpoints are saved to:
- Standard: `/workspace/shisa-v2.1/gpt2-125m/checkpoints/<run_name>/`
- MoE: `/workspace/shisa-v2.1/gpt2-125m/checkpoints/<experiment_name>/`

Each launch generates a timestamped folder (`dense_YYYYMMDD_HHMMSS` for dense runs, `moe_YYYYMMDD_HHMMSS` by default for MoE). Override the location with `RUN_NAME`, `RUN_TIMESTAMP`, or `CHECKPOINT_ROOT`. To intentionally replace an existing directory, export `OVERWRITE_CHECKPOINTS=1` before running the script.

### Real-time Monitoring

```bash
# Monitor training progress
docker exec <container_name> tail -f /workspace/shisa-v2.1/gpt2-125m/checkpoints/<run_name>/train.log

# Check GPU utilization
docker exec <container_name> watch -n 1 rocm-smi
```

## Best Practices

### Container Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shisa-ai/shisa-v2](https://github.com/shisa-ai/shisa-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
