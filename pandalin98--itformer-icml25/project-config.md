---
trigger: always_on
description: **ALWAYS follow these instructions first and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**
---

# ITFormer - Temporal-Textual Multimodal Question Answering Framework

**ALWAYS follow these instructions first and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

## Overview

ITFormer is a Python-based deep learning framework for temporal-textual multimodal question answering. It combines time series data with natural language processing using transformer architectures and large language models (Qwen2.5). The project requires GPU resources for optimal performance but can run on CPU for testing.

## Working Effectively

### Environment Setup and Dependencies

**NEVER CANCEL builds or installs - they may take 10+ minutes. Set timeout to 15+ minutes minimum.**

```bash
# Core Python dependencies - takes ~3-5 minutes. NEVER CANCEL.
pip install -r requirements.txt

# Additional required dependencies not in requirements.txt
pip install h5py nltk rouge-score scikit-learn
```

**Expected timing:**
- Main dependency installation: 3-5 minutes  
- Additional dependencies: 1-2 minutes
- **CRITICAL**: Set timeout to 15+ minutes for pip installs to avoid premature cancellation

### Project Structure Requirements

The project requires a specific directory structure to function correctly:

```
ITFormer-ICML25/
├── dataset/
│   └── datasets/                    # EngineMT-QA dataset files
│       ├── time_series_data.h5      # Time series data (required for inference)
│       ├── train_qa.jsonl           # Training QA pairs
│       └── test_qa.jsonl            # Test QA pairs (required for inference)
├── LLM/                             # Base LLM models directory
│   ├── Qwen2.5-0.5B-Instruct/       # For ITFormer-0.5B
│   ├── Qwen2.5-3B-Instruct/         # For ITFormer-3B  
│   └── Qwen2.5-7B-Instruct/         # For ITFormer-7B
├── checkpoints/                     # ITFormer model checkpoints
│   ├── ITFormer-0.5B/               # Lightweight model (beats ChatGPT-4o)
│   ├── ITFormer-3B/                 # Medium model
│   └── ITFormer-7B/                 # Best performance model
└── inference_results/               # Generated output directory
```

### Model and Dataset Downloads

**CRITICAL NETWORK REQUIREMENT**: Model downloads require internet access and may fail in restricted environments.

**Model download timing - NEVER CANCEL:**
- ITFormer-0.5B: ~15-30 minutes (smallest model)  
- ITFormer-7B: ~60-120 minutes (largest model)
- Qwen2.5 models: ~30-90 minutes each depending on size
- EngineMT-QA dataset: ~10-20 minutes

```bash
# Create required directories
mkdir -p LLM checkpoints dataset/datasets

# Install Git LFS for large model downloads  
git lfs install

# Download models (choose based on your needs - 0.5B recommended for testing)
# Set timeout to 120+ minutes for large model downloads
python -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='pandalin98/ITFormer-0.5B', local_dir='./checkpoints/ITFormer-0.5B')"
python -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='Qwen/Qwen2.5-0.5B-Instruct', local_dir='./LLM/Qwen2.5-0.5B-Instruct')"

# Download dataset
python -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='pandalin98/EngineMT-QA', repo_type='dataset', local_dir='./dataset/datasets')"
```

### Running Inference

**Primary inference methods:**

```bash
# Method 1: Direct Python execution (single GPU/CPU)
python inference.py --config yaml/infer.yaml

# Method 2: Multi-GPU with Accelerate (recommended for production)
# Takes ~5-30 minutes depending on dataset size and model. NEVER CANCEL.
accelerate launch --config_file accelerate_config.yaml inference.py --config yaml/infer.yaml

# Method 3: Using provided script
bash infra.bash
```

**Model size options:**
```bash
# ITFormer-0.5B (default) - fastest, good performance
python inference.py --config yaml/infer.yaml --model_checkpoint checkpoints/ITFormer-0.5B

# ITFormer-3B - medium performance  
python inference.py --config yaml/infer.yaml --model_checkpoint checkpoints/ITFormer-3B

# ITFormer-7B - best performance, slowest
python inference.py --config yaml/infer.yaml --model_checkpoint checkpoints/ITFormer-7B
```

**Expected inference timing - NEVER CANCEL:**
- Small dataset (100 samples): 2-5 minutes
- Medium dataset (1000 samples): 10-20 minutes  
- Full dataset (10k+ samples): 30-60 minutes
- **Set timeout to 90+ minutes for full inference runs**

### Validation and Testing

**Always run these validation steps after making changes:**

```bash
# 1. Test core imports (quick validation)
python -c "from models.TimeLanguageModel import TLM, TLMConfig; from dataset.dataset import TsQaDataset; from utils.metrics import open_question_metrics; print('✅ All modules import successfully')"

# 2. Test inference script help (validates argument parsing)
python inference.py --help

# 3. Syntax validation for all Python files
find . -name "*.py" -exec python -m py_compile {} \;

# 4. Test basic model initialization (requires models to be downloaded)
python -c "
import yaml
with open('yaml/infer.yaml', 'r') as f:
    config = yaml.safe_load(f)
print('✅ Configuration file loads correctly')
"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pandalin98/ITFormer-ICML25](https://github.com/Pandalin98/ITFormer-ICML25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
