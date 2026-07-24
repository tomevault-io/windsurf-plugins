---
trigger: always_on
description: `dicee` is a hardware-agnostic framework for training and using large-scale **Knowledge Graph Embedding (KGE)** models. Users train models on triples `(head, relation, tail)`, then query them for link prediction, multi-hop reasoning, and literal prediction.
---

# DICE Embeddings — Project Instructions

## Project Overview

`dicee` is a hardware-agnostic framework for training and using large-scale **Knowledge Graph Embedding (KGE)** models. Users train models on triples `(head, relation, tail)`, then query them for link prediction, multi-hop reasoning, and literal prediction.

**Key entry points:**
- CLI: `dicee --dataset_dir "KGs/UMLS" --model Keci`
- Python: `from dicee.executer import Execute; Execute(args).start()`
- Inference: `from dicee import KGE; model = KGE(path="...")`

---

## Architecture Pipeline

```
Input (dataset_dir | path_single_kg | sparql_endpoint)
  ↓ ReadFromDisk (dicee/read_preprocess_save_load_kg/read_from_disk.py)
  ↓ PreprocessKG  (dicee/read_preprocess_save_load_kg/preprocess.py)
     → entity_to_idx, relation_to_idx, er_vocab, re_vocab, ee_vocab
     → memory_map_train_set.npy
  ↓ construct_dataset() (dicee/dataset_classes/_factory.py)
     → torch.utils.data.Dataset per scoring technique
  ↓ DICE_Trainer (dicee/trainer/dice_trainer.py)
     → wraps TorchTrainer | TorchDDPTrainer | TensorParallel | PyTorch Lightning
  ↓ Evaluation (dicee/evaluation/evaluator.py)
     → MRR, MR, HITS@1, HITS@3, HITS@10
  ↓ Results stored under storage_path / path_to_store_single_run
```

---

## Models

All models extend `BaseKGE` in `dicee/models/base_model.py`. Required interface:

```python
class MyModel(BaseKGE):
    def __init__(self, args: dict): ...
    def forward_triples(self, x: torch.LongTensor) -> torch.FloatTensor: ...   # (B,3) → (B,)
    def forward_k_vs_all(self, x: torch.LongTensor) -> torch.FloatTensor: ...  # (B,2) → (B,|E|)
```

| File | Models |
|------|--------|
| `dicee/models/real.py` | DistMult, TransE, Pyke, Shallom, LFMult |
| `dicee/models/complex.py` | ComplEx, ConEx, AConEx |
| `dicee/models/quaternion.py` | QMult, ConvQ, AConvQ |
| `dicee/models/octonion.py` | OMult, ConvO, AConvO |
| `dicee/models/clifford.py` | Keci, CKeci, DeCaL, KeciTransformer |
| `dicee/models/function_space.py` | DualE |
| `dicee/models/transformers.py` | BytE, CoKE |
| `dicee/models/pykeen_models.py` | PykeenKGE (wraps any PyKEEN model) |

All models are exported from `dicee/models/__init__.py`.

**Clifford model convention:** `embedding_dim` must satisfy `embedding_dim / (p + q + 1) ∈ ℤ`.
The `args` dict passed to `__init__` comes from `config.Namespace.__dict__`.

---

## Trainers

| `--trainer` value | Class | Use Case |
|-------------------|-------|----------|
| `torchCPUTrainer` | `TorchTrainer` | CPU or single GPU |
| `PL` | PyTorch Lightning | Multi-GPU, recommended default |
| `torchDDP` | `TorchDDPTrainer` | Native DDP via `torchrun` |
| `TP` | `TensorParallel` | Tensor parallelism (1 model per GPU) — implements "Multiple Run Ensemble Learning with Low-Dimensional Knowledge Graph Embeddings" |

Multi-GPU with PL uses all visible CUDA devices automatically.
Use `CUDA_VISIBLE_DEVICES=0` to restrict to one GPU.
`--path_to_store_single_run` is required for DDP/multi-GPU runs.

---

## Scoring Techniques

| `--scoring_technique` | Dataset Class | Memory | Speed | Best For |
|-----------------------|---------------|--------|-------|----------|
| `NegSample` | `TriplePredictionDataset` | Low | Fast | Large KGs |
| `FixedNegSample` | `FixedNegSampleDataset` | Low | Fast | Continual learning |
| `1vsAll` | `OnevsAllDataset` | Medium | Medium | Small KGs |
| `KvsAll` | `KvsAll` | High | Slow | Default recommended |
| `KvsSample` | `KvsSampleDataset` | Medium | Medium | Balanced |
| `AllvsAll` | `AllvsAll` | Very high | Very slow | Pairwise loss |

Dataset classes live in `dicee/dataset_classes/`. All are routed through `construct_dataset()` in `dicee/dataset_classes/_factory.py`.

---

## Key Configuration Parameters (`dicee/config.py`)

### Data
| Param | Default | Description |
|-------|---------|-------------|
| `dataset_dir` | None | Folder with train.txt / valid.txt / test.txt |
| `path_single_kg` | None | Single RDF/OWL file |
| `sparql_endpoint` | None | SPARQL endpoint URL |
| `backend` | `"pandas"` | `pandas` \| `polars` \| `rdflib` |
| `separator` | `"\s+"` | Triple file column separator |

### Model
| Param | Default | Description |
|-------|---------|-------------|
| `model` | `"Keci"` | Model name string |
| `embedding_dim` | 64 | Embedding vector size |
| `p` | 0 | Clifford p-parameter |
| `q` | 1 | Clifford q-parameter |
| `input_dropout_rate` | 0.0 | Dropout on input embeddings |
| `hidden_dropout_rate` | 0.0 | Dropout on hidden layer |
| `normalization` | `"None"` | `"LayerNorm"` \| `"BatchNorm1d"` \| `"None"` |

### Training
| Param | Default | Description |
|-------|---------|-------------|
| `num_epochs` | 150 | Training epochs |
| `batch_size` | 1024 | Mini-batch size |
| `lr` | 0.1 | Learning rate |
| `optim` | `"Adam"` | `"Adam"` \| `"SGD"` \| `"ADOPT"` |
| `weight_decay` | 0.0 | L2 regularization |
| `neg_ratio` | 0 | Negatives per positive (NegSample) |
| `label_smoothing_rate` | 0.0 | Label smoothing coefficient |
| `trainer` | `"torchCPUTrainer"` | Trainer backend |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dice-group/dice-embeddings](https://github.com/dice-group/dice-embeddings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
