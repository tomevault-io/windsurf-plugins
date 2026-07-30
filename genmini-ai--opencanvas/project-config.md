---
trigger: always_on
description: The OpenCanvas API is a RESTful service for AI-powered presentation generation. It supports both topic-based and PDF-based generation with advanced features like image extraction and web research.
---

# OpenCanvas Generation API

## Overview

The OpenCanvas API is a RESTful service for AI-powered presentation generation. It supports both topic-based and PDF-based generation with advanced features like image extraction and web research.

## Key Features

🎨 **Dual Input Support**: Generate from text topics or PDF documents  
📸 **Image Extraction**: Extract and include images from PDF sources  
🔍 **Smart Research**: Automatic web research for insufficient knowledge  
🎯 **Multiple Themes**: Professional themes for different contexts  
📁 **Organized Output**: Structured file organization with timestamps  

## Architecture

```
src/api/
├── app.py           # FastAPI application setup
├── routes.py        # API endpoint definitions
├── services.py      # Business logic layer
├── models.py        # Pydantic request/response models
└── run_api.py       # Standalone server runner
```

**Dependencies**: The API imports core functionality from the `opencanvas` package:
- `opencanvas.generators.router` - Generation routing
- `opencanvas.config` - Configuration management

## Setup & Installation

### 1. Install Dependencies
```bash
# Core dependencies
pip install -r requirements.txt

# API-specific dependencies
pip install fastapi uvicorn

# Image extraction dependencies (recommended)
pip install docling docling-core pdfplumber

# Browser automation for PDF conversion
playwright install chromium
```

### 2. Configure Environment
```bash
# Copy and edit environment file
cp .env.example .env
# Edit .env with your API keys
```

Required API keys:
- `ANTHROPIC_API_KEY` - For generation (required)
- `BRAVE_API_KEY` - For web research (optional)

## Running the API

### Method 1: Standalone Runner (Recommended)
```bash
cd src
python api/run_api.py
```

### Method 2: Direct uvicorn
```bash
cd src
python -m uvicorn api.app:app --host 0.0.0.0 --port 8000 --reload
```

### Method 3: OpenCanvas CLI (if available)
```bash
opencanvas api --host 0.0.0.0 --port 8000 --reload
```

### API Access
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **Health Check**: http://localhost:8000/api/v1/health
- **Features**: http://localhost:8000/api/v1/features

## API Endpoints

### Generation Endpoint

#### `POST /api/v1/generate`
Generate presentation from topic or PDF source.

**Request Parameters:**
- `input_source`: Text topic or PDF URL/path
- `purpose`: Presentation purpose (academic, conference, corporate, etc.)
- `theme`: Visual theme (professional blue, clean minimalist, etc.)
- `extract_images`: Boolean - extract images from PDFs
- `output_dir`: Output directory (optional)

**Topic Generation Example:**
```json
{
  "input_source": "quantum computing basics",
  "purpose": "academic presentation",
  "theme": "professional blue"
}
```

**PDF Generation Example:**
```json
{
  "input_source": "https://arxiv.org/pdf/2301.08727.pdf",
  "purpose": "conference presentation",
  "theme": "clean minimalist",
  "extract_images": true
}
```

**Response:**
```json
{
  "success": true,
  "html_file": "output/quantum_computing_20250831_181245/slides/presentation.html",
  "research_performed": true,
  "organized_files": {
    "html": "output/quantum_computing_20250831_181245/slides/presentation.html",
    "sources": "output/quantum_computing_20250831_181245/sources/",
    "images": "output/quantum_computing_20250831_181245/images/"
  },
  "extracted_images_count": 3,
  "image_validation_report": {
    "total_images_checked": 5,
    "successful_replacements": 2
  },
  "message": "Presentation generated successfully"
}
```

### Configuration Endpoints

#### `GET /api/v1/themes`
Get available presentation themes.

#### `GET /api/v1/purposes`
Get available presentation purposes.

#### `GET /api/v1/features`
Check available features and dependencies.

**Response:**
```json
{
  "image_extraction": {
    "docling": true,
    "pdfplumber": true,
    "description": "Advanced image extraction from PDFs"
  },
  "supported": {
    "pdf_generation": true,
    "topic_generation": true,
    "html_to_pdf": true,
    "evaluation": true
  }
}
```

## Testing

### Basic Health Check
```bash
curl http://localhost:8000/api/v1/health
```

### Topic Generation Test
```bash
curl -X POST "http://localhost:8000/api/v1/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "input_source": "artificial intelligence ethics",
    "purpose": "academic presentation",
    "theme": "professional blue"
  }'
```

### PDF Generation with Image Extraction
```bash
curl -X POST "http://localhost:8000/api/v1/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "input_source": "https://arxiv.org/pdf/2301.08727.pdf",
    "purpose": "conference presentation", 
    "theme": "clean minimalist",
    "extract_images": true
  }'
```

### Concurrent Generation Testing
Test multiple generation requests simultaneously:
```bash
# Run in background
curl -X POST "http://localhost:8000/api/v1/generate" \
  -H "Content-Type: application/json" \
  -d '{"input_source": "quantum computing", "purpose": "academic presentation", "theme": "clean minimalist"}' \
  -o response1.json &


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genmini-ai/OpenCanvas](https://github.com/genmini-ai/OpenCanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
