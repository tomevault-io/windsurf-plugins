---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Endgame is a comprehensive machine learning toolkit providing 300+ estimators, transformers, and visualizers across tabular, time series, signal processing, CV, NLP, audio, and multimodal domains. It unifies state-of-the-art and classical methods under a consistent scikit-learn-compatible API for researchers and practitioners.

**Import Convention:** `import endgame as eg`

**Version:** 1.0.0 (see ROADMAP.md for detailed implementation status)

**Core Philosophy:** Classic-to-SOTA under one API. Sklearn-native, Polars-powered, interpretability-first. For research, production, and competition.

## Architecture

The library has the following core modules following the competition workflow:

```
eg.validation              → CV strategies, adversarial validation, drift detection
eg.preprocessing           → Feature engineering (Polars-based), encoding, class balancing
eg.models                  → 100+ models: GBDTs, trees, rules, Bayesian, neural, kernel, baselines
eg.ensemble                → Hill climbing, stacking, blending, threshold optimization
eg.calibration             → Conformal prediction, probability calibration, Venn-ABERS
eg.tune                    → Optuna integration with competition-specific search spaces
eg.explain                 → SHAP, LIME, PDP, feature interactions, counterfactuals
eg.fairness                → Demographic parity, equalized odds, bias mitigation
eg.anomaly                 → Isolation Forest, LOF, GritBot, PyOD integration
eg.semi_supervised         → Self-training for classification and regression
eg.benchmark               → Systematic evaluation, meta-learning, learning curves
eg.quick                   → One-line model training and comparison
eg.vision                  → timm backbones, TTA, WBF, augmentation pipelines
eg.nlp                     → Transformers, DAPT, pseudo-labeling, LLM utilities
eg.audio                   → Spectrogram conversion, SED models, audio augmentation
eg.timeseries              → Forecasting (statistical + neural), ROCKET classification
eg.signal                  → Filtering, spectral analysis, wavelets, entropy, complexity
eg.kaggle                  → Competition management, submissions, project scaffolding
eg.persistence             → Model save/load, ONNX export, model serving
eg.utils                   → Metrics, submission helpers, Sharpe ratio analysis
eg.automl                  → Intelligent AutoML framework (TabularPredictor, multi-modal)
eg.clustering              → 16 clustering algorithms with auto-selection
eg.dimensionality_reduction → PCA, UMAP, t-SNE, TriMAP, PHATE, VAE
eg.feature_selection       → 16+ methods: filter, wrapper, importance-based, advanced
eg.visualization           → 42 interactive HTML chart types
eg.tracking                → Experiment logging (MLflow, console)
eg.mcp                     → MCP server for LLM-powered ML pipelines
```

**Dependency Flow:** validation → preprocessing → models/vision/nlp/audio/signal/timeseries → ensemble → calibration

## Key Design Decisions

1. **Sklearn Interface**: All estimators implement `fit`/`predict`/`transform` for pipeline compatibility
2. **Polars-First**: Tabular preprocessing converts to `pl.LazyFrame` internally (accepts pandas/numpy input)
3. **Configuration Presets**: Competition-winning hyperparameters as defaults (e.g., `preset='endgame'`)
4. **Explicit Over Implicit**: No magic - every technique requires explicit invocation
5. **Lazy Loading**: Heavy modules (models, vision, nlp, audio, benchmark, kaggle, quick, visualization, persistence, explain, tracking, timeseries, signal, automl, dimensionality_reduction, feature_selection) loaded on demand

## Directory Structure

```
endgame/
├── core/                       # Base classes, Polars ops, config, types
├── validation/                 # AdversarialValidator, CV splitters (CPCV, StratifiedGroupKFold)
├── preprocessing/              # Encoders, aggregation, feature selection, DAE, imbalanced learning
├── models/
│   ├── wrappers.py             # Unified GBDT interface (LightGBM/XGBoost/CatBoost)
│   ├── trees/                  # Rotation Forest, C5.0/Cubist, Oblique, Quantile, Evolutionary
│   ├── rules/                  # RuleFit, FURIA
│   ├── bayesian/               # TAN, KDB, ESKDB, EBMC, AutoSLE, NeuralKDB
│   ├── tabular/                # FT-Transformer, SAINT, NODE, TabPFN, NAM, GANDALF, TabularResNet
│   ├── neural/                 # MLP, EmbeddingMLP, TabNet
│   ├── kernel/                 # GP, SVM
│   ├── baselines/              # ELM, NaiveBayes, LDA/QDA/RDA, KNN, Linear
│   ├── probabilistic/          # BART, NGBoost
│   ├── ordinal/                # Ordinal regression (mord wrappers)
│   ├── symbolic/               # PySR symbolic regression
│   ├── subgroup/               # PRIM bump hunting
│   └── ebm.py                  # Explainable Boosting Machines
├── ensemble/                   # Hill climbing, stacking, blending, threshold optimization
├── calibration/                # Conformal prediction, scaling methods, Venn-ABERS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allianceai/endgame](https://github.com/allianceai/endgame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
