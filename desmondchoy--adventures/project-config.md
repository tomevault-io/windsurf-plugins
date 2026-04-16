---
trigger: always_on
description: This file provides guidance to Gemini (Google AI) when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini (Google AI) when working with code in this repository.

## Project Overview

Learning Odyssey is an AI-powered interactive educational storytelling platform. Children select a story world and educational topic, then progress through personalized adventures where choices shape the narrative. The application uses LLM-generated content with structured chapter types (STORY, LESSON, REFLECT, CONCLUSION, SUMMARY).

## Development Commands

### Running the Application
```bash
# Activate virtual environment (required for all Python commands)
source .venv/bin/activate

# Run development server
uvicorn app.main:app --reload
```

### Dependencies
```bash
pip install -r requirements.txt
```

### Deployment: Cache Busting
When deploying code changes (especially JS/CSS), update version strings to force cache invalidation:

**Files with version strings:**
- `app/templates/base.html` — CSS files and `font-size-manager.js`
- `app/templates/components/scripts.html` — ES6 module (`main.js`)

**Version format:** `?v=YYYYMMDDC` (date + letter increment)
- Example: `?v=20260101a` → `?v=20260101b` → `?v=20260102a`

**Update all version strings when:**
- Modifying any JavaScript file
- Modifying any CSS file
- Fixing bugs that users might have cached

## Architecture Overview

### Core Flow
1. **WebSocket Router** (`app/routers/websocket_router.py`) — Entry point for adventure sessions, handles JWT auth
2. **AdventureStateManager** (`app/services/adventure_state_manager.py`) — Central state management
3. **ChapterManager** (`app/services/chapter_manager.py`) — Chapter type sequencing and story element selection
4. **LLMServiceFactory** (`app/services/llm/factory.py`) — Dual-model architecture for cost optimization

### Dual-Model LLM Architecture
The factory pattern routes tasks to appropriate models (~50% cost reduction):
- **Gemini Flash** (29% of ops): Complex reasoning — `story_generation`, `image_scene_generation`
- **Gemini Flash Lite** (71% of ops): Simple processing — `summary_generation`, `paragraph_formatting`, `character_visual_processing`, `image_prompt_synthesis`

```python
from app.services.llm.factory import LLMServiceFactory
llm = LLMServiceFactory.create_for_use_case("story_generation")  # Returns Flash
llm = LLMServiceFactory.create_for_use_case("summary_generation")  # Returns Flash Lite
```

### State Model
`AdventureState` (in `app/models/story.py`) is the central data structure:
- `planned_chapter_types`: Pre-determined sequence of ChapterType enums
- `chapters`: List of completed ChapterData objects
- `selected_narrative_elements`, `selected_sensory_details`: Story elements chosen at initialization
- `chapter_summaries`, `lesson_questions`: Accumulated data for final summary
- `character_visuals`: Dict tracking visual descriptions for all characters (for image consistency)
- `protagonist_description`: Base visual description of protagonist
- `metadata`: Agency details, challenge history, timing data

### Chapter Type Sequencing
Configurable chapter length (currently 10) with rules:
- First chapter through second-to-last chapter: eligible for STORY, LESSON, or REFLECT
- Last chapter: always CONCLUSION
- First chapter is always STORY (with agency choice)
- One LESSON-REFLECT-STORY sequence placed randomly in the middle chapters
- Two additional non-consecutive LESSON chapters
- Remaining positions: STORY

### WebSocket Services (`app/services/websocket/`)
- `core.py`: Connection management and coordination
- `choice_processor.py`: User choice handling, triggers character visual updates
- `content_generator.py`: Chapter content generation
- `stream_handler.py`: Live streaming responses (chunk-by-chunk for performance)
- `image_generator.py`: AI image generation with two-step prompt synthesis
- `summary_generator.py`: Final adventure summary

### Data Sources
- `app/data/lessons/*.csv`: Educational questions by topic
- `app/data/stories/*.yaml`: Story categories with narrative elements, sensory details, themes

### Persistence
- **Supabase**: User auth (Google OAuth + Guest), adventure state, telemetry
- `StateStorageService`: State persistence with RLS policies
- `TelemetryService`: Analytics logging with duration tracking

## Critical Development Patterns

### 1. Dynamic Narrative Handling (CRITICAL)
Narrative content is generated dynamically by LLMs and is inherently variable. **Never hardcode narrative text in application logic or tests.**
- Rely on structure (AdventureState fields, ChapterType) not specific content
- Use metadata for reliable decision-making
- Tests should verify state transitions and structure, not narrative sentences

### 2. Validated Streaming Pattern
Use `stream_chapter_with_live_generation()` to generate, validate, then stream:
```python
# CORRECT — Collect and validate first, then stream approved content
chapter_content = await generate_chapter_content_with_retries(...)
await stream_text_content(chapter_content.content, websocket)
await websocket.send_json({"type": "choices", "choices": ...})
```
This intentionally prioritizes correctness:
- Story chapters must have exactly 3 validated choices before content is shown
- Unvalidated live token streaming should be avoided for chapter output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DesmondChoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
