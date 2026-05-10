---
trigger: always_on
description: This file is the living memory for Claude Code sessions on this project.
---

# CLAUDE.md — Project Memory & Tracking
# arabic-itsm-classification

This file is the living memory for Claude Code sessions on this project.
Update it continuously as decisions are made, experiments are run, and milestones are reached.
It will serve as a primary source for the university post-project documentation report.

---

## Assistant Operating Baseline

- Follow repository-local instructions in `AGENTS.md` as the primary operating policy.
- Keep experiment reporting reproducible by anchoring claims to notebook outputs and files under `results/`.
- Archive each full pipeline analysis using naming: `analysis_nb<NN>_<env>_<date>.md` (env: `local_cpu`, `local_gpu`, `kaggle_gpu`).

---

## Project Identity

| Field            | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Title**        | Cloud-Based ITSM Ticket Classification Platform Using Fine-Tuned Transformer Models |
| **Author**       | Mohamed A. Elbaz                                           |
| **Supervisor**   | Dr. Eman E. Sanad, Assistant Professor of IT, FCAI, Cairo University |
| **Degree**       | Professional Master's in Cloud Computing Networks                    |
| **Institution**  | Faculty of Computers and Artificial Intelligence, Cairo University    |
| **Date Started** | February 2026                                                        |
| **Repo**         | arabic-itsm-classification                                           |
| **Dataset Repo** | [bazokhan/arabic-itsm-dataset](https://github.com/bazokhan/arabic-itsm-dataset) |

---

## Dataset Summary

| Property          | Value                                                   |
|-------------------|---------------------------------------------------------|
| Source (GitHub)   | https://github.com/bazokhan/arabic-itsm-dataset         |
| Source (HF)       | https://huggingface.co/datasets/albaz2000/arabic-itsm-dataset |
| Size              | 10,000 tickets                                          |
| Dialect           | Egyptian Arabic (عامية مصرية) with EN code-mixing       |
| Generation        | LLM-generated (Gemini), validated programmatically      |
| L1 categories     | 6 (Access, Network, Hardware, Software, Security, Service) |
| L2 categories     | 16                                                     |
| L3 categories     | 48                                                     |
| Labels            | category_level_1/2/3, priority (1-5), sentiment        |
| Format            | CSV + JSONL                                             |

---

## Architecture Decisions

### ADR-001 — Primary Model: MarBERTv2
- **Date**: February 2026
- **Decision**: Use `UBC-NLP/MARBERTv2` as the primary encoder
- **Rationale**: Best-in-class for Egyptian colloquial Arabic; pretrained on Twitter (dialect-heavy, noisy); validated Feb 2025 benchmarks show 64–85% F1 on Egyptian classification tasks; low compute cost vs ByT5
- **Alternatives considered**: CAMeLBERT (good but MSA-biased), AraBERTv2 (MSA-only), ByT5 (excellent noise handling but high compute)
- **Reference**: `docs/model_recommendation.md`
- **Status**: Accepted

### ADR-002 — Classification Strategy: Multi-Head, Start with L1
- **Date**: February 2026
- **Decision**: Train separate classification heads for L1 (6 classes), L2 (16 classes), L3 (48 classes), and priority (5 classes). Prioritize L1 first, then extend using the same pipeline structure.
- **Rationale**: Flat multiclass gets sparse at deeper levels after deduplication. Hierarchical modeling mirrors ITSM routing logic and keeps interpretation clean.
- **Status**: Accepted

### ADR-003 — Framework: HuggingFace Transformers + PyTorch
- **Date**: February 2026
- **Decision**: Use `transformers` + `torch` with `Trainer` API for fine-tuning
- **Rationale**: Native support for MarBERTv2; standard in academic NLP work; easy to export to ONNX/TorchScript for deployment
- **Status**: Accepted

### ADR-004 — Data Split: Stratified 70/15/15
- **Date**: February 2026
- **Decision**: Stratified split on L1 label — 70% train, 15% validation, 15% test
- **Rationale**: Maintains class distribution across splits; 1,500 test samples sufficient for reliable metric estimation; validation set sized for hyperparameter search
- **Status**: Accepted

### ADR-005 — Dataset Deduplication Applied in Preprocessing, Not at Source
- **Date**: February 2026
- **Decision**: Remove 451 exact duplicate (title_ar, description_ar) pairs during preprocessing in Notebook 02, not by modifying the source dataset CSV on GitHub/HuggingFace.
- **Rationale**: Duplicates create data leakage when identical texts land in both train and test splits, artificially inflating all metrics. Fixing in preprocessing rather than at source preserves the raw dataset as a stable, citable artefact while keeping the transformation explicit and reproducible. This is standard ML practice and must be documented in the methodology chapter of the thesis.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bazokhan/arabic-itsm-classification](https://github.com/bazokhan/arabic-itsm-classification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
