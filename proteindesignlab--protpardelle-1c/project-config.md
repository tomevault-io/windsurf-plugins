---
trigger: always_on
description: Concise, project-specific guidance to help an AI coding agent be productive quickly. Focus on THESE repo conventions (not generic ML advice).
---

## Protpardelle-1c AI Assistant Instructions

Concise, project-specific guidance to help an AI coding agent be productive quickly. Focus on THESE repo conventions (not generic ML advice).

### 1. High-level Architecture

Protpardelle-1c is a diffusion-based protein structure (and sequence) generative framework.

Main layers:

- `src/protpardelle/core/` - Modeling logic
  - `models.py` defines composite model class `Protpardelle` plus submodules: `CoordinateDenoiser` (U-ViT/DiT style) and optional `MiniMPNN` for sequence scoring / design; orchestrates conditioning (motifs, hotspots, sidechain/backbone crop conditions) and sampling loops.
  - `diffusion.py` noise schedules + coordinate perturbation utilities. Noise level increases with timestep; keep this monotonic increasing convention consistent when adding schedules.
  - `modules.py` contains reusable NN building blocks (attention, resblocks, positional encodings incl. chain-relative and rotary variants) and learning rate scheduler `LinearWarmupCosineDecay`.
- `src/protpardelle/data/` - Data abstractions
  - `pdb_io.py` feature extraction from PDB/mmCIF -> tensors and PDB writers (`load_feats_from_pdb`, `write_coords_to_pdb`). Respect `chain_residx_gap` logic when generating multi-chain residues.
  - `dataset.py` dataset + batching utilities (`PDBDataset`, cropping / random rotations, `make_fixed_size_1d`).
  - `motif.py` parses contig spec strings (RFdiffusion-like) into motif placement indices.
  - `atom.py` masks and conversions between backbone-only and full atom (37) representations.
- `src/protpardelle/integrations/` - External model hooks (ESMFold eval, ProteinMPNN sequence design) - loaded lazily based on flags (`num_mpnn_seqs`).
- `src/protpardelle/common/` - Static biochemical constants (`residue_constants.py`) plus lightweight protein data containers (`protein.py`: `Protein`, `Hetero`, `PDB_CHAIN_IDS`) reused across I/O, sampling, training and PDB serialization (`to_pdb`).
- `src/protpardelle/sample.py` - CLI (Typer) sampling orchestrator; builds search space combos, runs model sampling, optional ProteinMPNN sequence design, evaluation (self-consistency) and writes results under `results/` (or `PROTPARDELLE_OUTPUT_DIR`).
- `src/protpardelle/train.py` - CLI training loop with mixed precision + (optionally) `nn.DataParallel`.
- `src/protpardelle/evaluate.py` - Sequence design and self-consistency utilities.


### 2. Configuration & Execution

- Configs live under `examples/sampling/*.yaml` and `examples/training/*.yaml`. They are not Hydra multilevel packages; sampling builds Cartesian products over `search_space` lists manually (see `sample.py`). Maintain parameter names exactly (e.g. `step_scales`, `schurns`, `crop_cond_starts`).
- Environment variables (see README) override auto-detected paths: `PROTPARDELLE_MODEL_PARAMS`, `PROTEINMPNN_WEIGHTS`, `ESMFOLD_PATH`, `PROTPARDELLE_OUTPUT_DIR`, `FOLDSEEK_BIN`.
- Model checkpoints + configs stored under `model_params/` (subfolders `configs/` + `weights/`). Loading expects a config name matching weight stems (e.g. `cc58_epoch416.pth` with `configs/cc58.yaml`).

### 3. Sampling Workflow (critical path)

1. Read motif / input structure (if any) via `load_feats_from_pdb` -> features dict.
2. Build search space (product of user lists) -> each combination calls model inference.
3. Diffusion loop uses noise schedule in `diffusion.py`; `schurn` injects extra stochasticity (scaled internally by step count).
4. Crop / motif / hotspot conditioning implemented via coordinate masking (see `apply_crop_cond_strategy` in `models.py`). Sidechain tip conditioning uses curated atom lists in `residue_constants.RFDIFFUSION_BENCHMARK_TIP_ATOMS`.
5. Optional sequence design: ProteinMPNN (`integrations/protein_mpnn.py`) or MiniMPNN head (sequence diffusion / self-conditioning modes `seqdes`).
6. Write PDBs: for backbone-only samples first convert with `bb_coords_to_atom37_coords` if needed; chain IDs resolved either by provided mapping or sequential from `'A'` upward.

Key invariants: ensure `residue_index` starts at 1 post normalization; apply `add_chain_gap` only once; keep shape conventions `(B, N, A, 3)` for coords, `(B, N)` for indices/masks.

### 4. Training Workflow

- Entry: `python -m protpardelle.train <model_name> <output_dir>` (often submitted via `scripts/train.sbatch`).
- Datasets assembled from YAML config fields referencing prepared AI-CATH / interface data (see README dataset section). Sampling noise schedule functions: `uniform`, `lognormal`, `mpnn`, `constant` (must match those in `diffusion.noise_schedule`).
- Losses: masked MSE for coordinates (`masked_mse_loss`) + optionally cross-entropy for sequence tokens (`masked_cross_entropy_loss`). When adding a new loss term, wire it in inside the training loop where `total_loss` is accumulated (search for existing accumulation pattern). Maintain masking semantics (divide by sum(mask) with clamp >=1e-6).

### 5. Data & Tensor Conventions

- Atom ordering fixed by `residue_constants.atom_order`; backbone indices stored in `bb_idxs` in `CoordinateDenoiser`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProteinDesignLab/protpardelle-1c](https://github.com/ProteinDesignLab/protpardelle-1c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
