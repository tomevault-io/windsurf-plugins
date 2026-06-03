---
trigger: always_on
description: **quackML** is a DuckDB extension implementing a full-service AI/ML engine written in Rust. It enables developing AI/ML models in pure SQL with DuckDB, bringing models to data rather than vice versa.
---

# CLAUDE.md - AI Assistant Guide for quackML

## Project Overview

**quackML** is a DuckDB extension implementing a full-service AI/ML engine written in Rust. It enables developing AI/ML models in pure SQL with DuckDB, bringing models to data rather than vice versa.

### Supported Capabilities
- **Traditional ML**: Linear/logistic regression, XGBoost, LightGBM, SVM, random forests, clustering, decomposition
- **Deep Learning**: HuggingFace Transformers integration for text tasks
- **NLP Tasks**: Text classification, text generation, summarization, translation, question answering, embeddings
- **Fine-tuning**: Support for fine-tuning models like GPT-2 on custom data

## Codebase Architecture

```
quackML/
├── src/
│   ├── lib.rs              # Extension entrypoint, registers SQL functions
│   ├── api.rs              # SQL function implementations (train, finetune, predict, etc.)
│   ├── bindings/           # ML library bindings
│   │   ├── mod.rs          # Bindings trait definition
│   │   ├── linfa.rs        # Pure Rust ML (linear/logistic regression, SVM)
│   │   ├── xgboost.rs      # XGBoost bindings
│   │   ├── lightgbm.rs     # LightGBM bindings
│   │   ├── sklearn/        # Scikit-learn Python bindings
│   │   ├── transformers/   # HuggingFace Transformers bindings
│   │   ├── python/         # Python/venv management
│   │   └── langchain/      # LangChain integration
│   ├── context/            # Database connection context management
│   │   └── context.rs      # Global DATABASE_CONTEXT for connection access
│   ├── orm/                # Data models and database operations
│   │   ├── mod.rs          # Re-exports all ORM types
│   │   ├── project.rs      # Project management (ML projects)
│   │   ├── model.rs        # Model storage and deployment
│   │   ├── snapshot.rs     # Data snapshots for training
│   │   ├── dataset.rs      # Dataset loading and management
│   │   ├── algorithm.rs    # Supported algorithm enum
│   │   ├── task.rs         # ML task types (regression, classification, etc.)
│   │   ├── metrics.rs      # Model evaluation metrics
│   │   ├── sampling.rs     # Train/test sampling strategies
│   │   ├── search.rs       # Hyperparameter search
│   │   ├── strategy.rs     # Deployment strategies
│   │   └── status.rs       # Job status tracking
│   └── sql/
│       └── schema.sql      # Database schema (tables, views, enums)
├── duckdb-rs/              # Git submodule: forked duckdb-rs with extension support
├── Cargo.toml              # Rust dependencies and workspace config
├── requirements.txt        # Python dependencies
└── *.csv                   # Sample datasets (iris, digits, diabetes, etc.)
```

## Key Modules Explained

### `src/lib.rs` - Extension Entry Point
- Defines `quack_ml_init()` as the DuckDB extension entrypoint
- Initializes database context and schema
- Registers all SQL functions: `train`, `finetune`, `predict`, `predict_text`, `predict_proba`, `embed`, `transform`, `generate`, `load_dataset`

### `src/api.rs` - SQL Function Implementations
- Implements DuckDB virtual table functions (`VTab`) for `train` and `finetune`
- Implements scalar functions for predictions and embeddings
- Handles parameter binding and result generation
- Large file (~35k tokens) - read in chunks if needed

### `src/bindings/` - ML Library Integrations

**Core Trait** (`mod.rs`):
```rust
pub trait Bindings: Send + Sync + Debug {
    fn predict(&self, features: &[f32], num_features: usize, num_classes: usize) -> Result<Vec<f32>>;
    fn predict_proba(&self, features: &[f32], num_features: usize) -> Result<Vec<f32>>;
    fn to_bytes(&self) -> Result<Vec<u8>>;
    fn from_bytes(bytes: &[u8]) -> Result<Box<dyn Bindings>>;
}
```

**Implementations**:
- `linfa.rs`: Pure Rust implementations (LinearRegression, LogisticRegression, Svm)
- `xgboost.rs`: XGBoost gradient boosting
- `lightgbm.rs`: LightGBM gradient boosting
- `sklearn/mod.rs`: Scikit-learn via PyO3 (many algorithms)
- `transformers/mod.rs`: HuggingFace models via PyO3

### `src/orm/` - Data Models

**Key Types**:
- `Project`: ML project container (name, task type)
- `Model`: Trained model with hyperparams, metrics, serialized bindings
- `Snapshot`: Frozen dataset for reproducible training
- `Dataset`: In-memory training/test data splits
- `Algorithm`: Enum of all supported algorithms (49 variants)
- `Task`: ML task types (regression, classification, text_classification, etc.)

### `src/context/context.rs` - Database Context
- Global `DATABASE_CONTEXT` for connection access
- `context::run(|conn| ...)` pattern for database operations

## SQL Schema (`src/sql/schema.sql`)

**Tables**:
- `quackml.projects`: ML project definitions
- `quackml.models`: Trained models with metrics and hyperparams
- `quackml.snapshots`: Data snapshots for training
- `quackml.deployments`: Model deployment history
- `quackml.logs`: Training logs
- `quackml.files`: Serialized model file storage

**Custom Types**:
- `task`: regression, classification, text_classification, embedding, etc.
- `sampling`: random, stratified, time_series, last
- `strategy`: new_score, best_score, most_recent, rollback, specific

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parkerdgabel/quackML](https://github.com/parkerdgabel/quackML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
