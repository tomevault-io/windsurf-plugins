---
trigger: always_on
description: The Multi-Vision Toolkit is a standalone Python application designed for the local execution of advanced vision models, including Florence-2, Janus-Pro-1B, and Qwen2.5-VL. It offers a graphical user interface for tasks such as image captioning, object detection, and OCR, facilitating the creation of AI training datasets.
---

# Gemini Technical Reference Manual - Multi-Vision Toolkit

## 1. System Synopsis

The Multi-Vision Toolkit is a standalone Python application designed for the local execution of advanced vision models, including Florence-2, Janus-Pro-1B, and Qwen2.5-VL. It offers a graphical user interface for tasks such as image captioning, object detection, and OCR, facilitating the creation of AI training datasets.

### 1.1 Core Capabilities
- **Model Integration**: Supports Florence-2, Janus-Pro-1B, Qwen2.5-VL, and Qwen3-VL-4B-Instruct.
- **Graphical Interface**: A Tkinter-based GUI with features like drag-and-drop and batch processing.
- **Prompt Engineering**: A secure templating system for dynamic prompt generation.
- **Resource Optimization**: Includes automatic GPU memory management and quantization.
- **Dataset Curation**: A workflow for sorting images into `approved` and `rejected` sets for AI training.

## 2. System Architecture

### 2.1 Application Core
- **`main.py:1-166`**: Handles application initialization, environment checks, and GUI startup.
- **`main.py:30-86`**: Contains the environment validation logic, specifically addressing and mitigating flash attention conflicts.

### 2.2 Model Subsystem (`models/`)
The model subsystem is built on a base class for consistency and extensibility.

- **`base_model.py:37-50`**: The `BaseVisionModel` ABC, which includes device optimization logic.
- **`florence_model.py`**: Implements Microsoft's Florence-2 model.
- **`janus_model.py`**: Implements DeepSeek's Janus-Pro-1B model.
- **`qwen_model.py`**: Implements Alibaba's Qwen2.5-VL with quantization.
- **`qwen3_model.py`**: Implements Alibaba's Qwen3-VL-4B-Instruct model.
- **`qwen_model_local.py`**: An offline-optimized variant of the Qwen model.

**Contingency Models:**
- **`dummy_florence_model.py`**: A fallback for Florence-2 loading failures.
- **`dummy_janus_model.py`**: A CLIP-based fallback for the Janus model.
- **`dummy_qwen_model.py`**: A basic fallback for the Qwen models.
- **`dummy_qwen3_model.py`**: A fallback for Qwen3 model loading failures.

### 2.3 Prompt Engineering Subsystem (`templates/`)
This subsystem provides secure and manageable prompt templating.

- **`template_manager.py`**: Manages template loading and path validation.
- **`template_engine.py`**: Handles variable substitution with built-in security against injection.
- **`default_templates.json`**: Contains default templates for the integrated models.
- **`user_templates.json`**: Stores user-defined templates.

### 2.4 Data Flow & Storage
- **`data/review/`**: Ingress directory for images awaiting classification.
- **`data/approved/`**: Egress directory for images approved for dataset inclusion.
- **`data/rejected/`**: Egress directory for rejected images.
- **`models/weights/`**: Designated storage for local model weights.

## 3. Developer Onboarding

### 3.1 Environment Configuration

**System Prerequisites:**
```bash
# Set up a Conda environment using Python 3.11
conda create -n vision-env python=3.11
conda activate vision-env

# Install the specified PyTorch version for system stability
pip install torch==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/cu126
```

**Project Dependencies (`requirements.txt:1-28`):**
- **Core**: PyTorch 2.6.0, a git version of Transformers for Qwen support.
- **Vision Libraries**: Pillow, OpenCV, timm, einops.
- **GUI**: tkinterdnd2 for drag-and-drop.
- **Performance**: bitsandbytes for quantization, accelerate for GPU optimization.

### 3.2 VRAM Allocation & Optimization

**VRAM Thresholds:**
- **Florence-2 Large**: Requires >=8GB VRAM.
- **Janus-Pro-1B**: Requires >=4GB VRAM.
- **Qwen2.5-VL-3B**: Requires >=6GB VRAM with 8-bit quantization.
- **Qwen3-VL-4B-Instruct**: Requires >=10GB VRAM (standard) or less with quantization.

**Optimization Directives:**
```bash
# De-allocate GPU memory
python clear_gpu_memory.py

# Monitor VRAM usage
nvidia-smi

# Set environment variables to enforce quantization and memory policies
export QWEN_FORCE_4BIT=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
```

### 3.3 Security Protocols

The prompt engineering subsystem has several security controls.

**Input Sanitization:**
- Variable names are validated against a whitelist in `template_engine.py`.
- Path traversal is mitigated in `template_manager.py`.
- All user-facing inputs are subject to HTML escaping and length restrictions.

**Secure Code Patterns:**
```python
# Variable name whitelist in template_engine.py
ALLOWED_VARIABLE_NAMES = {
    'trigger_word', 'image_context', 'quality_mode', 
    'task_type', 'question', 'focus', 'model_name', 'filename'
}

# Path validation in template_manager.py
templates_dir = Path(templates_dir).resolve()
if not str(templates_dir).startswith(str(Path(__file__).parent.parent)):
    raise ValueError("Invalid templates directory - potential path traversal detected")
```

## 4. Standard Operating Procedures

### 4.1 Integrating a New Vision Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Limbicnation/multi-vision-toolkit](https://github.com/Limbicnation/multi-vision-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
