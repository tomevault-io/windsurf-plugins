---
trigger: always_on
description: Reference for AI agents working in this repo. Keep it open and follow it literally — commands, paths, and config keys are exact.
---

# CLAUDE.md — D-FINE-seg agent guide

Reference for AI agents working in this repo. Keep it open and follow it literally — commands, paths, and config keys are exact.

## 1. What this repo is

D-FINE-seg is a detection + instance segmentation framework built on D-FINE. A single config (`config.yaml`, Hydra-based) drives the whole pipeline: dataset split → train → export → bench → infer. One task flag (`task: detect` or `task: segment`) switches between object detection and instance segmentation.

Main supported model sizes: `n`, `s`, `m`, `l`, `x`. Pretrained weights live in `pretrained/` (`dfine_<size>_coco.pt` and `dfine_<size>_obj2coco.pt`).

## 2. Layout

```
config.yaml                  # main Hydra config (edit this for most tasks)
Makefile                     # thin wrappers around python -m src.dl.*
pretrained/                  # dfine_{n,s,m,l,x}_{coco,obj2coco}.pt — must exist before training
src/
  etl/                       # dataset prep: split, yolo2coco, coco2yolo, polys2bbox, …
  dl/                        # train.py, export.py, bench.py, infer.py, validator.py, ov_int8.py, …
  d_fine/                    # model architecture (backbone, encoder, decoder, matcher, losses)
  infer/                     # multi-backend inference wrappers (torch, onnx, ov, trt, coreml, litert)
```

## 3. Environment

- Python 3.11–3.13, PyTorch 2.9, CUDA 12.x. Dependencies live in [pyproject.toml](pyproject.toml); [uv.lock](uv.lock) is the source of truth for versions.
- Install with `uv sync` (creates `.venv/`). All Makefile targets shell out via `uv run`, so no manual activation is needed for `make train` / `make bench` / etc. For ad-hoc commands either prefix with `uv run` or activate the venv (`source .venv/bin/activate`).
- Platform-specific deps are gated by markers in `pyproject.toml`: `tensorrt` installs on Linux only. `coremltools` ships wheels for both platforms (Linux can run the converter for `make export`, even though the CoreML runtime itself is macOS-only). `uv.lock` covers both so the same lockfile works on the dev mac and the lab box.
- Pretrained weights auto-download from Hugging Face (`ArgoSA/D-FINE-seg`) into `pretrained/` on first use via `ensure_pretrained` in [src/d_fine/utils.py](src/d_fine/utils.py). Triggered from `build_model` in [src/d_fine/dfine.py](src/d_fine/dfine.py) only when the filename matches `dfine_<size>_<dataset>.pt`; custom checkpoint paths still raise `FileNotFoundError` if missing.

## 4. Configuration model

All CLI commands use Hydra, so any config key is overridable on the command line with dotted paths:

```bash
python -m src.dl.train exp_name=my_exp model_name=s train.batch_size=12 train.epochs=50
```

Key top-level fields in [config.yaml](config.yaml):

| Field | Meaning |
|---|---|
| `project_name` | WandB project name |
| `exp_name` | Experiment name (outputs nest under `<exp_name>_<date>`) |
| `model_name` | `n` / `s` / `m` / `l` / `x` |
| `task` | `detect` or `segment` |
| `train.root` | Absolute project root (dataset + outputs live here) |
| `train.data_path` | Dataset dir — `${train.root}/data/dataset` by default |
| `train.coco_dataset` | `False` → YOLO-style; `True` → COCO JSON |
| `train.pretrained_dataset` | `coco` or `obj2coco` |
| `train.pretrained_model_path` | Path to init weights (swap this to fine-tune from a custom checkpoint) |
| `train.path_to_save` | Where `model.pt`, `last.pt`, logs, and configs land |
| `train.path_to_test_data` | Folder of images/videos for `infer.py` |
| `train.label_to_name` | 0-indexed, contiguous class map |
| `train.ddp.enabled` / `train.ddp.n_gpus` | Multi-GPU switch |
| `train.conf_thresh` / `train.iou_thresh` | Detection thresholds |

LRs are indexed by model size under `train.lrs.<size>.{backbone_lr, base_lr}`.

Preset dataset configs in [configs/](configs/) can be used as templates — copy one to `config.yaml` and edit paths / classes.

## 5. Dataset preparation

### 5.1 YOLO layout (default)

```
<train.data_path>/
  images/   # .jpg/.png/.jpeg
  labels/   # .txt — same stem as image
```

Label format:
- **detect**: `class_id xc yc w h` (normalized, cxcywh)
- **segment**: `class_id x1 y1 x2 y2 … xN yN` (normalized polygon)

Generate splits:

```bash
make split        # == python -m src.etl.split
```

Produces `train.csv`, `val.csv` (and `test.csv` if `split.val_split < 1 - split.train_split`) inside `train.data_path`. Ratios live under the top-level `split:` section in `config.yaml`.

### 5.2 COCO layout

```
<train.data_path>/
  images/
  train.json
  val.json
  test.json         # optional
```

Then set `train.coco_dataset: True`. No `make split` needed.

### 5.3 Conversion / cleanup utilities

In [src/etl/](src/etl/): `yolo2coco.py`, `coco2yolo.py`, `polys2bbox.py`, `png_mask_to_yolo.py`, `remove_dups.py`, `clean_csv.py`, `split_from_yolo.py`. Run as `python -m src.etl.<name>`.

## 6. Training

### 6.1 Single-GPU

```bash
make train
# or explicit:
python -m src.dl.train
# with overrides:
python -m src.dl.train exp_name=fine_s model_name=s task=detect train.batch_size=12 train.epochs=30
```

### 6.2 Multi-GPU (DDP)

Set `train.ddp.enabled: True` and `train.ddp.n_gpus: N` in `config.yaml`, then:

```bash
make train

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArgoHA/D-FINE-seg](https://github.com/ArgoHA/D-FINE-seg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
