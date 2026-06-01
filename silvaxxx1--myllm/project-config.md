---
trigger: always_on
description: **MyLLM** is a from-scratch LLM framework built for deep understanding and research.
---

# CLAUDE.md — MyLLM Project Intelligence

## 🧠 Project Overview

**MyLLM** is a from-scratch LLM framework built for deep understanding and research.
It covers the full pipeline: tokenization → attention → training → RLHF → inference.

**Three-layer architecture:**
- `notebooks/` — guided, from-first-principles learning (21 notebooks)
- `Modules/` — isolated, focused experiments (data, models, training, finetuning, inference)
- `myllm/` — production-grade core framework (SFT, DPO, PPO, quantization, REST API)

**Primary focus:** The `myllm/` core framework — everything else serves as reference and experimentation ground for it.

---

## 📁 Project Structure

```
MyLLM/
├── notebooks/          # 21 learning notebooks (0.0 → 6.4 + Appendices)
├── Modules/
│   ├── 1.data/         # Tokenizers, dataloader, preprocessor
│   ├── 2.models/       # GPT, LLaMA2, LLaMA3.2, attention variants (MHA/MQA/GQA/Flash)
│   ├── 3.training/     # Training loops, distributed training
│   ├── 4.finetuning/   # SFT (spam, instruction), DPO, PPO, QLoRA
│   └── 5.inference/    # GPT2 inference app, quantization
├── demos/              # 5 Colab-ready notebooks (install → quickstart → SFT)
├── myllm/              # Core framework (installable package)
│   ├── __init__.py     # Full public API — LLM, ModelConfig, trainers, tokenizers
│   ├── __main__.py     # CLI: python -m myllm version / models / info
│   ├── model.py        # Core LLM definition
│   ├── api.py          # LLM class (load, generate, generate_text, generate_batch)
│   ├── Configs/        # ModelConfig, GenerationConfig (dataclasses)
│   ├── configs/        # Lowercase alias → from myllm.configs import ModelConfig
│   ├── Tokenizers/     # GPT2, LLaMA2, LLaMA3, trainable tokenizer
│   ├── tokenizers/     # Lowercase alias → from myllm.tokenizers import GPT2Tokenizer
│   ├── Train/          # SFT, DPO, PPO trainers + Engine
│   ├── train/          # Lowercase alias → from myllm.train import SFTTrainer
│   └── utils/          # Loaders, samplers, weight mappers, model registry
├── models/             # Auto-downloaded weights cache (GPT2 small/medium/large/xl)
└── main.py
```

---

## 🛠 Tech Stack

- **Language:** Python 3.10+
- **Core ML:** PyTorch 2.x (pure — no HuggingFace abstractions in core)
- **Experiment tracking:** Weights & Biases (wandb)
- **Tokenizers:** tiktoken (GPT2), SentencePiece (LLaMA2), custom trainable
- **Inference UI:** Gradio
- **API:** FastAPI
- **Package manager:** uv (see `uv.lock`) — prefer `uv` over `pip`
- **Config:** YAML + Python dataclasses

---

## ⚙️ Environment Setup

```bash
# Preferred: use uv
uv sync

# Or pip
pip install -r requirements.txt

# Install as editable package
pip install -e .
```

---

## 🔑 Key Conventions

### Code Style
- Pure PyTorch — avoid HuggingFace model abstractions in `myllm/` core
- Every module should be readable standalone — no hidden magic
- Prefer explicit over implicit: name dimensions, document tensor shapes
- Use type hints throughout

### Naming
- Model configs live in `myllm/Configs/` (dataclasses)
- Trainer configs live in `myllm/Train/configs/`
- Test files are prefixed `test_` and live in `myllm/tests/`
- Checkpoint dirs follow pattern: `output_{experiment_name}/checkpoint-{step}/`

### Tensor Shape Comments
Always annotate tensor shapes in model code:
```python
# x: (batch, seq_len, d_model)
x = self.attn(x)  # (batch, seq_len, d_model)
```

---

## 🚀 Common Commands

### Run a training experiment
```bash
python Modules/3.training/train.py --config configs/basic.yml
```

### Load a model and generate
```python
from myllm import LLM, GenerationConfig

llm = LLM.from_pretrained("gpt2-small")   # config + weights + tokenizer in one call
result = llm.generate_text("Hello world", GenerationConfig(max_length=50), skip_prompt=True)
print(result["text"])
```

### Run the core framework SFT trainer
```python
from myllm import ModelConfig
from myllm.train import SFTTrainer, SFTTrainerConfig

trainer = SFTTrainer(
    SFTTrainerConfig(output_dir="./output", report_to=[]),
    model_config=ModelConfig.from_name("gpt2-small"),
)
trainer.setup_model()
trainer.setup_data(train_dataloader=my_dataloader)
trainer.setup_optimizer()
trainer.train()
```

### Serve the REST API
```bash
python myllm/api.py
```

### Run all tests
```bash
uv run pytest
```

### Run specific test suites
```bash
uv run pytest myllm/tests/test_e2e.py        # end-to-end pipeline
uv run pytest myllm/tests/test_model.py      # model components
uv run pytest myllm/tests/test_training.py   # trainers
uv run pytest myllm/tests/test_api.py        # inference API
uv run pytest -x                             # stop on first failure
uv run pytest -q                             # quiet output
```

### Benchmark inference
```bash
python myllm/benchmark_api.py
```

---

## 📚 Notebook Learning Path

Follow notebooks in order for full understanding:

| Stage | Notebooks | Topic |
|-------|-----------|-------|
| 0 | `0.0.WELCOME` | Orientation |
| 1 | `1.1` → `1.2` | Data & Tokenization |
| 2 | `2.1` → `2.4` | Attention & Transformer Architectures |
| 3 | `3.1` → `3.2` | Training & Advanced Training |
| 4 | `4.1` → `4.3` | Supervised Fine-Tuning (SFT, PEFT/LoRA) |
| 5 | `5.1` → `5.2` | RLHF (PPO) & DPO |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silvaxxx1/MyLLM](https://github.com/silvaxxx1/MyLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
