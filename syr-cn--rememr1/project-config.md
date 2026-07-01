---
trigger: always_on
description: **Paper:** "Look Back to Reason Forward: Revisitable Memory for Long-Context LLM Agents" (ICLR 2026)
---

# ReMemR1 — Revisitable Memory for Long-Context LLM Agents

**Paper:** "Look Back to Reason Forward: Revisitable Memory for Long-Context LLM Agents" (ICLR 2026)
**Authors:** Yaorui Shi, An Zhang, Xiang Wang et al.

ReMemR1 trains LLM agents to process long documents chunk-by-chunk while maintaining and revisiting an evolving memory. The agent learns through GRPO-based RL with a multi-level reward combining outcome correctness (EM/F1) and step-level state advantages (format + action rewards for memory updates/recalls).

## Architecture Overview

The system has three pillars:
1. **Recurrent Agent** — Chunk-by-chunk document processing with memory update/recall actions
2. **Multi-Level Reward** — Outcome reward + state advantage, mixed by alpha (default 0.8)
3. **Training via verl** — GRPO advantage, Ray distributed training, SGLang rollout, FSDP

## Directory Structure

```
ReMemR1/
  recurrent/                    # Core recurrent agent framework
    interface.py                # Abstract bases: RAgent, RDataset, RConfig, RRegister, AsyncRAgent
    generation_manager.py       # Sync generation loop orchestrator
    async_generation_manager.py # Async generation loop orchestrator
    utils.py                    # TokenTemplate, chat_template(), pad/unpad, final_batch()
    tool.py                     # Tool interface
    async_utils.py              # ChatCompletionProxy for async rollout
    impls/                      # Concrete implementations
      memory_revisit.py         # MemoryAgent, MemoryConfig, MemoryDataset, REGISTER
      tf_idf_retriever.py       # TfidfRetriever (sklearn-based)
      async_memory.py           # Async version of memory agent
    chat_template/              # Chat template utilities
    test/                       # Unit tests and perf benchmarks
  verl/                         # Modified verl RL training framework
    trainer/
      main_ppo.py               # Training entry point (hydra config)
      ppo/
        ray_trainer.py          # Main training loop, reward aggregation, advantage computation
        core_algos.py           # PPO/GRPO core algorithms, KL controllers
        metric_utils.py         # Format/action/data metrics, validation metrics
    workers/                    # Distributed worker implementations
      actor/                    # Actor (policy) workers
      critic/                   # Critic workers
      rollout/                  # Rollout workers (SGLang/vLLM)
      reward_manager/           # Reward computation managers
      reward_model/             # Reward model workers
      sharding_manager/         # FSDP sharding
    utils/
      reward_score/
        hotpotqa.py             # Outcome reward: EM/F1 for QA tasks
        math.py, gsm8k.py, ... # Other reward functions
      dataset/rl_dataset.py     # Base RLHFDataset
      checkpoint/               # FSDP/Megatron checkpoint managers
    models/                     # Model implementations (Qwen2, LLaMA, Megatron)
    protocol.py                 # DataProto — the universal data container
  taskutils/
    memory_eval/                # Evaluation pipeline
      run_eval.py               # Main eval entry point
      utils/
        rememr1.py              # Inference: async_query_llm() for ReMemR1 evaluation
        tf_idf_retriever.py     # TF-IDF retriever (eval copy)
        envs.py                 # Eval env config (URL, API_KEY, chunk sizes)
    data_synthesis/             # Data preprocessing scripts
  scripts/
    0_run_data_process.sh       # Download and preprocess benchmarks
    1_run_train_ReMemR1_7B.sh   # Train ReMemR1-7B (reference config)
    1_run_train_ReMemR1_3B.sh   # Train ReMemR1-3B
    2_run_eval_ReMemR1.sh       # Run evaluation
    merge_ckpt.sh               # Merge FSDP checkpoints
    model_merger.py             # Model merging utility
    converter_hf_to_mcore.py    # HF to Megatron-Core conversion
```

## Key Abstractions

### RAgent / RConfig / RDataset / RRegister (`recurrent/interface.py`)

The plugin system for recurrent agents. All custom implementations register via `RRegister`:

```python
REGISTER = RRegister(config_cls=MemoryConfig, dataset_cls=MemoryDataset, agent_cls=MemoryAgent)
```

- **RConfig** — Dataclass base for agent config. Subclass adds domain-specific fields.
- **RDataset** — Extends `RLHFDataset`. Override `__getitem__` and `get_bactch_keys` (note: typo in codebase, keep it).
- **RAgent** — Lifecycle: `start()` -> loop of `action()` -> rollout -> `update()` until `done()` -> `end()`. Returns `(final_mask, sample_index)` from `end()`.
- **AsyncRAgent** — Alternative async interface with `rollout()` method for single-sample async generation.
- **RRegister.from_filename()** — Dynamic loading of implementations from file path + object name.

### MemoryAgent (`recurrent/impls/memory_revisit.py`)

The core ReMemR1 agent (402 lines). Key design:

- **Chunk processing:** Iterates over context chunks (sized `config.chunk_size` tokens). Active samples tracked via `active_mask = ctx_length > step * chunk_size`.
- **Memory state:** `self.memory` (current memory per sample), `self.history_memory` (set of all past memories per sample), `self.recall_memories` (last recalled memory).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syr-cn/ReMemR1](https://github.com/syr-cn/ReMemR1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
