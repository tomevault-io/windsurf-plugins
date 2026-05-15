---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project creates long-form horror story videos for YouTube using a 5-phase automated pipeline. The system integrates existing AI tools (Kokoro-FastAPI, StableDiffusion-Local, N8N, ObsidianKnowledgeBase) to generate 1-2+ hour videos with narration, atmospheric imagery, and background audio. Target: 7 videos per weekly batch production day.

## Development Commands

### Phase 0: YouTube Transcript Analysis
```bash
# Extract transcripts from successful horror story channels (no video download)
python scripts/content_analyzer.py --channels horror_channels.json --extract-transcripts --min-views 100000

# Analyze story patterns using Fabric AI
python scripts/content_analyzer.py --fabric-analysis --pattern extract_story_hooks --transcripts research/transcripts/
python scripts/content_analyzer.py --fabric-analysis --pattern analyze_pacing_structure --transcripts research/transcripts/
python scripts/content_analyzer.py --fabric-analysis --pattern extract_horror_elements --transcripts research/transcripts/

# Generate story templates from successful transcript patterns
python scripts/inspiration_generator.py --transcripts-analysis research/fabric_analysis/ --create-templates

# Correlate story patterns with performance metrics  
python scripts/inspiration_generator.py --correlate-performance --views-threshold 500000 --output research/high_performing_patterns.json
```

### Phase 1: Story Preparation
```bash
# Create story script from inspiration brief
python scripts/story_processor.py --inspiration research/inspiration_briefs/brief_001.json --output ~/Claude/YouTube-Content/horror-stories/

# Generate image prompts and timing from story script
python scripts/story_processor.py --script ~/Claude/YouTube-Content/horror-stories/story_001/script.json --generate-prompts

# Validate story structure and pacing
python scripts/story_processor.py --validate ~/Claude/YouTube-Content/horror-stories/story_001/
```

### Phase 2: Asset Generation
```bash
# Generate narration audio using Kokoro-FastAPI
python scripts/asset_generator.py --narration --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Generate atmospheric images using StableDiffusion-Local
python scripts/asset_generator.py --images --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Generate complete asset library for story
python scripts/asset_generator.py --all --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Quality check generated assets
python scripts/asset_generator.py --validate ~/Claude/YouTube-Content/horror-stories/story_001/assets/
```

### Phase 3: Video Assembly
```bash
# Assemble final video with ffmpeg
python scripts/video_assembler.py --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Preview video with timing validation
python scripts/video_assembler.py --preview --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Generate video with custom transitions
python scripts/video_assembler.py --story story_001 --transition-style fade --duration-per-image 8
```

### Phase 4: Publishing
```bash
# Generate YouTube metadata and thumbnails
python scripts/publisher.py --metadata --story ~/Claude/YouTube-Content/horror-stories/story_001/

# Upload to YouTube with scheduling
python scripts/publisher.py --upload --story story_001 --schedule "2024-01-01 20:00"

# Batch upload multiple videos
python scripts/publisher.py --batch-upload ~/Claude/YouTube-Content/horror-stories/ready_for_upload/
```

### Testing and Development
```bash
# Run full pipeline test with sample story
python -m pytest tests/ -v

# Test individual phase
python -m pytest tests/test_story_processor.py::test_inspiration_integration -v

# Validate complete pipeline end-to-end
python scripts/pipeline_validator.py --test-story sample_horror_story
```

## External Tool Integration

### yt-dlp (YouTube Content Analysis)
- **Purpose**: Extract transcripts and metadata from successful horror channels
- **Installation**: `pip install yt-dlp`
- **Usage**: Transcript-only extraction (no video download) for story pattern analysis
- **Integration**: Subprocess calls from content_analyzer.py

### Fabric AI (Content Pattern Analysis)
- **Purpose**: Analyze transcripts for story structures, hooks, pacing patterns
- **Location**: Connected to local Ollama models  
- **Patterns**: Custom patterns for horror story analysis (extract_story_hooks, analyze_pacing_structure)
- **Integration**: Direct API calls for transcript processing

### Kokoro-FastAPI (Text-to-Speech)
- **Endpoint**: `http://localhost:8880/v1/audio/speech`
- **Docker**: `docker run -d -p 8880:8880 --name kokoro-full ghcr.io/remsky/kokoro-fastapi-cpu:latest`
- **Features**: 67 voice models, voice mixing, multiple formats (MP3/WAV)
- **Integration**: REST API calls from asset_generator.py

### StableDiffusion-Local (Image Generation)  
- **Location**: `~/Claude/StableDiffusion-Local/`
- **Command**: `python generate.py "prompt" --cpu --seed X --width 512 --height 288`
- **Performance**: 45-second generation, horror-capable DreamShaper-8 model
- **Integration**: CLI subprocess calls with batch processing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Costa318/YouTube-StoryVideo-Generator](https://github.com/Costa318/YouTube-StoryVideo-Generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
