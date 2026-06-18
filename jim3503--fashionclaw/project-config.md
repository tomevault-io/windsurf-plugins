---
trigger: always_on
description: description: Virtual try-on system for clothing. Generate realistic images of models wearing specific outfits using AI. Supports single and batch processing with Gemini 3.1 Flash Image Preview model.
---

---
name: fashionclaw
description: Virtual try-on system for clothing. Generate realistic images of models wearing specific outfits using AI. Supports single and batch processing with Gemini 3.1 Flash Image Preview model.
metadata: {"clawdbot":{"emoji":"👔","requires":{"bins":["python3"],"py":["requests","PIL"]}}}
---

# FashionClaw Virtual Try-On 👔

Professional virtual try-on system powered by Google Gemini 3.1 Flash Image Preview model. Generate high-quality images of models wearing specific clothing items.

## Features

- **Fast Generation**: ~22 seconds per image (65% faster than competitors)
- **High Quality**: 896×1195 resolution output
- **Batch Processing**: Process multiple try-on operations at once
- **Flexible Operations**: Edit mode, style transfer, and image generation
- **Detailed Reporting**: Complete performance and quality metrics

## Prerequisites

- Python 3.8+
- OpenRouter API key with access to Gemini models
- Dependencies: `requests`, `Pillow` (PIL), `numpy`

Install dependencies:
```bash
pip install requests pillow numpy
```

## Directory Structure

```
/home/ming/ai-projects/outfitanyone/
├── model/          # Model images (1-1.jpg to 1-10.jpg)
├── cloth/          # Clothing images (1-1.jpg to 1-5.jpg)
└── outputs/        # Generated results (auto-created)
```

## Setup

Set your OpenRouter API key:

```bash
export OPENROUTER_API_KEY="your_api_key_here"
```

Or store it in `~/.outfitanyone/config.json`.

## Quick Batch Processing

Edit `/home/ming/.agents/skills/outfitanyone/example_batch.json` to configure your model-clothing pairs:

```json
{
  "pairs": [
    ["/home/ming/ai-projects/outfitanyone/model/1-1.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-1.jpg"],
    ["/home/ming/ai-projects/outfitanyone/model/1-2.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-2.jpg"],
    ["/home/ming/ai-projects/outfitanyone/model/1-3.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-3.jpg"]
  ],
  "output_dir": "/home/ming/ai-projects/outfitanyone/outputs"
}
```

Then run batch processing:

```bash
cd /home/ming/.agents/skills/outfitanyone
python virtual_tryon_skill.py --batch example_batch.json
```

## Usage Examples

### Basic Virtual Try-On

```python
import sys
sys.path.append('/home/ming/.agents/skills/outfitanyone')

from virtual_tryon_skill import VirtualTryOnSkill

skill = VirtualTryOnSkill()

# 单张虚拟试穿
result = skill.try_on(
    model_image="/home/ming/ai-projects/outfitanyone/model/1-1.jpg",
    cloth_image="/home/ming/ai-projects/outfitanyone/cloth/1-1.jpg",
    output_path="/home/ming/ai-projects/outfitanyone/outputs/result.png"
)

if result["success"]:
    print(f"Generated: {result['output_path']}")
    print(f"Time: {result['generation_time']}s")
```

### Batch Processing (JSON Config)

Create or edit `batch_config.json`:

```json
{
  "pairs": [
    ["/home/ming/ai-projects/outfitanyone/model/1-1.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-1.jpg"],
    ["/home/ming/ai-projects/outfitanyone/model/1-2.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-2.jpg"],
    ["/home/ming/ai-projects/outfitanyone/model/1-3.jpg", "/home/ming/ai-projects/outfitanyone/cloth/1-3.jpg"]
  ],
  "output_dir": "/home/ming/ai-projects/outfitanyone/outputs"
}
```

Then run:

```python
import sys
sys.path.append('/home/ming/.agents/skills/outfitanyone')
import json
from virtual_tryon_skill import VirtualTryOnSkill

skill = VirtualTryOnSkill()

# Load config from JSON
with open('/home/ming/.agents/skills/outfitanyone/batch_config.json', 'r') as f:
    config = json.load(f)

pairs = config["pairs"]
output_dir = config.get("output_dir", "/home/ming/ai-projects/outfitanyone/outputs")

results = skill.batch_try_on(
    pairs=pairs,
    output_dir=output_dir
)

# Print results
for i, result in enumerate(results, 1):
    if result["success"]:
        print(f"{i}. ✅ {result['output_path']}")
    else:
        print(f"{i}. ❌ {result['error']}")
```

### Command Line Usage

```bash
# Single image
python virtual_tryon_skill.py \
  --model /home/ming/ai-projects/outfitanyone/model/1-1.jpg \
  --cloth /home/ming/ai-projects/outfitanyone/cloth/1-1.jpg \
  --output /home/ming/ai-projects/outfitanyone/outputs/result.png

# Batch processing using JSON config
python virtual_tryon_skill.py \
  --batch /home/ming/.agents/skills/outfitanyone/example_batch.json \
  --output-dir /home/ming/ai-projects/outfitanyone/outputs

# Style transfer mode
python virtual_tryon_skill.py \
  --model /home/ming/ai-projects/outfitanyone/model/1-1.jpg \
  --cloth /home/ming/ai-projects/outfitanyone/cloth/1-1.jpg \
  --operation style_transfer \
  --output /home/ming/ai-projects/outfitanyone/outputs/style_result.png
```

## Parameters

### Main Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model_image` | str | Required | Path to model photo |
| `cloth_image` | str | Required | Path to clothing image |
| `output_path` | str | Auto-generated | Output image path |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jim3503/FashionClaw](https://github.com/Jim3503/FashionClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
