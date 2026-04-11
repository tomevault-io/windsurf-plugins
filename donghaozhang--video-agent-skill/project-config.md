---
trigger: always_on
description: **⚠️ ALWAYS start with FREE tests before ANY paid content generation**
---

# Development Guidelines for AI Content Generation

## CRITICAL: Cost Protection First

**⚠️ ALWAYS start with FREE tests before ANY paid content generation**

Every development workflow must begin with:
1. **FREE environment tests** (`test_setup.py`)
2. **FREE API connection tests** (`test_api_only.py` for video)
3. **Only then proceed** to paid generation with explicit user confirmation

## Project Architecture

### Multi-Platform AI Content Generation
This project provides comprehensive AI content generation capabilities:

1. **Video Generation** (Google Veo + FAL AI Dual-Model)
2. **Avatar Generation** (FAL AI Triple-Mode with lip-sync)
3. **Text-to-Image Generation** (FAL AI Quad-Model)
4. **Text-to-Speech Generation** (ElevenLabs TTS Package with modular architecture)

### Directory Structure Standards
```
<content_type>_<platform>/
├── <platform>_<content_type>_generator.py  # Main generator class
├── demo.py                                 # Cost-conscious interactive demo
├── test_setup.py                          # FREE environment validation
├── test_<specific>.py                     # PAID generation tests
├── requirements.txt                       # Dependencies
├── .env                                   # API configuration
├── README.md                              # Complete documentation
├── output/                                # Generated content
└── test_output/                           # Test-generated content
```

### Implementation Standards

#### Generator Class Architecture
All generator classes follow consistent patterns:
```python
class <Platform><ContentType>Generator:
    def __init__(self, api_key: Optional[str] = None):
        """Initialize with API key from .env or parameter"""
        
    def generate_<content_type>(self, **kwargs) -> Dict[str, Any]:
        """Universal generation method with model/mode selection"""
        
    def generate_<content_type>_with_<specific_model>(self, **kwargs) -> Dict[str, Any]:
        """Model-specific optimized methods"""
        
    def test_connection(self) -> bool:
        """FREE API connectivity test"""
```

#### Cost-Conscious Testing Framework
Every module must include:
1. **FREE Tests**: Environment, dependencies, API connectivity
2. **PAID Tests**: Actual content generation with cost warnings
3. **Official Examples**: Documentation compliance (for avatar generation)
4. **Interactive Demos**: User-friendly testing with cost confirmations

## Implementation Patterns

### FAL AI Video Generation (Dual-Model)
**Location**: `fal_video_generation/`
**Models**: MiniMax Hailuo-02, Kling Video 2.1
**Key Features**: 
- Universal methods with model selection
- Model-specific optimization methods
- Prompt optimization (Hailuo), CFG scale and negative prompts (Kling)

```python
# Universal interface with model selection
generator.generate_video_from_image(
    prompt="Description",
    image_url="path/to/image.jpg",
    model="hailuo",  # or "kling"
    duration="6"
)

# Model-specific optimization
generator.generate_video_with_hailuo(prompt_optimizer=True)
generator.generate_video_with_kling(cfg_scale=0.7, negative_prompt="blur")
```

### FAL AI Avatar Generation (Triple-Mode)
**Location**: `fal_avatar_generation/`
**Modes**: Text-to-Speech, Audio-to-Avatar, Multi-Audio Conversation
**Key Features**:
- 20 voice options with official example compliance
- Natural lip-sync and expression generation
- Conversation support with seamless transitions
- Official FAL AI example reproduction

```python
# Text-to-Speech Mode (20 voices available)
generator.generate_avatar_video(
    image_url="path/to/image.jpg",
    text_input="Your text here",
    voice="Bill",  # Official example default
    num_frames=136,  # Official example frames
    seed=42,  # Official example seed
    turbo=True
)

# Audio-to-Avatar Mode
generator.generate_avatar_from_audio(
    image_url="path/to/image.jpg",
    audio_url="path/to/audio.mp3",
    num_frames=145  # Default for audio mode
)

# Multi-Audio Conversation Mode
generator.generate_multi_avatar_conversation(
    image_url="path/to/image.jpg",
    first_audio_url="path/to/person1.mp3",
    second_audio_url="path/to/person2.mp3",
    num_frames=181  # Default for multi-audio mode
)

# Official Example Reproduction
generator.generate_official_example()  # Exact documentation compliance
```

### FAL AI Text-to-Image Generation (Quad-Model)
**Location**: `fal_text_to_image/`
**Models**: Imagen4, Seedream, FLUX Schnell, FLUX Dev
**Key Features**:
- Batch generation with multiple models
- Cost-conscious design with confirmation prompts
- Dragon generation for testing scenarios

```python
# Single model generation
generator.generate_image(
    prompt="A beautiful dragon",
    model="imagen4",
    negative_prompt="blur, artifacts"
)

# Batch generation with multiple models
generator.batch_generate(
    prompt="A magical forest",
    models=["imagen4", "flux_schnell"],
    auto_confirm=False  # Cost confirmation required
)
```

### ElevenLabs Text-to-Speech Generation (Modular Package)
**Location**: `text_to_speech/`
**Features**: Professional modular architecture, 3000+ voices, AI content pipeline
**Key Benefits**: Clean imports, comprehensive TTS capabilities, OpenRouter AI integration

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/donghaozhang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
