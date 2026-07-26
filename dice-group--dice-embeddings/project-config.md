---
trigger: always_on
description: `dicee` is a hardware-agnostic framework for training and using large-scale **Knowledge Graph Embedding (KGE)** models. Users train models on triples `(head, relation, tail)`, then query them for link prediction, multi-hop reasoning, and literal prediction.
---

# dicee — Claude Code Instructions

`dicee` is a hardware-agnostic framework for training and using large-scale **Knowledge Graph Embedding (KGE)** models. Users train models on triples `(head, relation, tail)`, then query them for link prediction, multi-hop reasoning, and literal prediction.

**Entry points:**
- CLI: `dicee --dataset_dir "KGs/UMLS" --model Keci`
- Python training: `from dicee.executer import Execute; Execute(args).start()`
- Inference: `from dicee import KGE; model = KGE(path="...")`

Git workflow: PRs target `develop`, not `main` (see `CONTRIBUTING.md`).

---

## Architecture Pipeline

```
Input (dataset_dir | path_single_kg | sparql_endpoint)
  → ReadFromDisk       dicee/read_preprocess_save_load_kg/read_from_disk.py
  → PreprocessKG       dicee/read_preprocess_save_load_kg/preprocess.py
     → entity_to_idx, relation_to_idx, er_vocab, re_vocab, ee_vocab, memory_map_train_set.npy
  → construct_dataset()  dicee/dataset_classes/_factory.py
     → torch.utils.data.Dataset per scoring technique
  → DICE_Trainer       dicee/trainer/dice_trainer.py
     → wraps TorchTrainer | TorchDDPTrainer | TorchFSDPTrainer | TensorParallel | PyTorch Lightning
  → Evaluator          dicee/evaluation/evaluator.py (real implementation; dicee/evaluator.py is a re-export shim — edit the former)
     → MRR, MR, HITS@1, HITS@3, HITS@10
  → Results under storage_path / path_to_store_single_run
```

## Models

All models extend `BaseKGE` in `dicee/models/base_model.py` (`forward_triples` at line ~466, `forward_k_vs_all` at ~484). Required interface:

```python
class MyModel(BaseKGE):
    def __init__(self, args: dict): ...
    def forward_triples(self, x: torch.LongTensor) -> torch.FloatTensor: ...   # (B,3) → (B,)
    def forward_k_vs_all(self, x: torch.LongTensor) -> torch.FloatTensor: ...  # (B,2) → (B,|E|)
```

| File | Models |
|------|--------|
| `dicee/models/real.py` | DistMult, TransE, Shallom, Pyke, CoKE (+ CoKEConfig) |
| `dicee/models/complex.py` | ComplEx, ConEx, AConEx |
| `dicee/models/quaternion.py` | QMult, ConvQ, AConvQ |
| `dicee/models/octonion.py` | OMult, ConvO, AConvO |
| `dicee/models/clifford.py` | Keci, CKeci, DeCaL, KeciTransformer |
| `dicee/models/function_space.py` | FMult, GFMult, FMult2, LFMult1, LFMult |
| `dicee/models/dualE.py` | DualE |
| `dicee/models/pykeen_models.py` | PykeenKGE (wraps any PyKEEN model) |
| `dicee/models/transformers.py` | BytE (**not** exported from `models/__init__.py`; import directly) |

`dicee/models/__init__.py` star-imports real/complex/quaternion/octonion/pykeen_models/function_space and explicitly imports `Keci, CKeci, DeCaL, KeciTransformer` and `DualE`. `transformers.py`, `literal.py`, `ensemble.py`, and `fsdp_models.py` are **not** re-exported — import those submodules directly.

**Clifford model convention:** `embedding_dim / (p + q + 1)` must be an integer. The `args` dict passed to `__init__` comes from `vars(config.Namespace())`.

## Trainers

| `--trainer` | Class | File | Use Case |
|-------------|-------|------|----------|
| `torchCPUTrainer` | `TorchTrainer` | `torch_trainer.py` | CPU or single GPU |
| `PL` | `lightning.pytorch.Trainer` | — | Multi-GPU, recommended default |
| `torchDDP` | `TorchDDPTrainer` | `torch_trainer_ddp.py` | Native DDP via `torchrun` |
| `torchFSDP` | `TorchFSDPTrainer` | `torch_trainer_fsdp.py` | Fully-sharded data parallel |
| `TP` | `TensorParallel` | `model_parallelism.py` | Tensor parallelism (1 model/GPU) — "Multiple Run Ensemble Learning with Low-Dimensional KGE" |

Note: the dependency is the `lightning` package (`import lightning.pytorch as pl`), not the older standalone `pytorch-lightning`.

Multi-GPU with `PL` uses all visible CUDA devices automatically; restrict with `CUDA_VISIBLE_DEVICES=0`. `--path_to_store_single_run` is required for DDP/FSDP/multi-GPU runs.

## Scoring Techniques

Dataset classes live in `dicee/dataset_classes/` (`_negative_sampling.py`, `_label_based.py`, `_literal.py`, `_bpe.py`), all routed through `construct_dataset()` in `_factory.py`.

| `--scoring_technique` | Dataset Class | Notes |
|-----------------------|---------------|-------|
| `NegSample` | `TriplePredictionDataset` | Large KGs; set `--neg_ratio` |
| `FixedNegSample` | `FixedNegSampleDataset` | Continual learning; stable negatives |
| `1vsAll` | `OnevsAllDataset` | Small KGs |
| `KvsAll` | `KvsAll` | Default recommended |
| `KvsSample` | `KvsSampleDataset` | Balanced memory/speed |
| `AllvsAll` | `AllvsAll` | Full pairwise; very slow, avoid on large KGs |
| `1vsSample` | `OnevsSample` | Sampled 1-vs-all |
| `FSDP1vsSample` | `FSDP1vsSampleDataset` | FSDP-compatible variant |

## Key Configuration (`dicee/config.py`)

| Param | Default | Notes |
|-------|---------|-------|
| `dataset_dir` / `path_single_kg` / `sparql_endpoint` | `None` | Pick one input source |
| `backend` | `"pandas"` | `pandas` \| `polars` \| `rdflib` |
| `separator` | `"\s+"` | Triple file column separator |
| `model` | `"Keci"` | Model name string |
| `embedding_dim` | 64 | |
| `p`, `q` | 0, 1 | Clifford params |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dice-group/dice-embeddings](https://github.com/dice-group/dice-embeddings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
