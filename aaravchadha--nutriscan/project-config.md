---
trigger: always_on
description: NutriScan is an AI-powered nutrition assistant that combats food insecurity through personalized dietary insights. It reads nutrition labels (vision AI, with OCR as offline fallback), identifies food from photos (vision AI), generates nutritious recipes from available ingredients, and connects users to free/low-income food resources nearby. Built by 3 Purdue freshmen (Aarav, Neil, Nuv) through Dataception for the undergraduate research symposium on **April 16, 2026**. All-free tech stack.
---

# NutriScan Build Plan

## Context
NutriScan is an AI-powered nutrition assistant that combats food insecurity through personalized dietary insights. It reads nutrition labels (vision AI, with OCR as offline fallback), identifies food from photos (vision AI), generates nutritious recipes from available ingredients, and connects users to free/low-income food resources nearby. Built by 3 Purdue freshmen (Aarav, Neil, Nuv) through Dataception for the undergraduate research symposium on **April 16, 2026**. All-free tech stack.

## Tech Stack
- **Python** + **Streamlit** (pure Python frontend)
- **Groq Vision API** with Llama 4 Scout `meta-llama/llama-4-scout-17b-16e-instruct` — **primary** label reader and food photo identifier. Takes an image, returns structured JSON (nutrition fields / food items). Replaced the original Tesseract OCR path for Upload Label and Recipe "Add from Label" flows after Phase 4.2 testing showed OCR was unreliable on real iPhone photos.
- **Tesseract** via `pytesseract` + **OpenCV** — offline **fallback** for the label reader when `GROQ_API_KEY` is missing or the vision API errors out. Kept intact so the app degrades gracefully, not used on the golden path.
- **Groq Text API** with Llama 3.3-70b-versatile — LLM analysis of nutrition data + health profile → allergen/preservative/nutrient flags, goal alignment, recommendations (free tier)
- **USDA FoodData Central API** (free key from api.data.gov) — primary source for raw/whole foods
- **Open Food Facts API** (no key, no rate limits) — fallback for branded packaged products that USDA misses
- **pillow-heif** — HEIC/HEIF support so iPhone photos open natively in the file uploader without client-side conversion
- **Food resource locator API** (TBD — USDA Food Desert Atlas, FoodFinder, Feeding America, or 211.org; fallback: curated local list)
- API keys in `.env` (gitignored), `.env.example` committed for teammates

## Project Structure
```
NutriScan/
├── app.py                      # Streamlit entry point
├── requirements.txt
├── .env.example                # GROQ_API_KEY=  USDA_API_KEY=
├── .env                        # actual keys (gitignored)
├── .gitignore
├── src/
│   ├── ocr/                    # Fallback label reader (Phase 3.1)
│   │   ├── preprocessor.py     # OpenCV image preprocessing (EXIF, normalize width, adaptive threshold)
│   │   └── extractor.py        # Tesseract OCR + regex parsing
│   ├── llm/
│   │   ├── groq_client.py      # Groq API wrapper
│   │   └── prompts.py          # Prompt templates (analysis, recipe, food vision, label vision)
│   ├── nutrition/
│   │   ├── models.py           # Dataclasses
│   │   ├── fda_guidelines.py   # DV% computation
│   │   ├── usda_client.py      # USDA API client + lookup_food fallback wrapper
│   │   └── openfoodfacts_client.py  # Open Food Facts API client (fallback)
│   ├── vision/                 # Primary label + food readers (Phase 3.4, Phase 4.2)
│   │   ├── food_identifier.py  # Groq vision → food items + USDA/OFF bridge
│   │   └── label_reader.py     # Groq vision → NutritionData (replaces OCR on golden path)
│   ├── resources/
│   │   └── locator.py          # Free food resource finder (food banks, pantries, etc.)
│   └── ui/
│       ├── components.py       # Reusable Streamlit widgets
│       ├── pages_upload.py     # Image upload tab
│       ├── pages_snap.py       # Snap Food photo tab
│       ├── pages_manual.py     # Manual entry tab
│       ├── pages_recipe.py     # Recipe Generator tab
│       ├── pages_find.py       # Find Free Food Near You tab
│       └── pages_results.py    # Results display
├── data/
│   └── fda_daily_values.json   # FDA daily reference values
├── tests/
│   ├── test_ocr.py
│   ├── test_llm.py
│   ├── test_nutrition.py
│   └── sample_labels/
└── eval/
    ├── ocr_accuracy.py
    └── ground_truth.json
```

## Key Packages
```
streamlit>=1.30.0
opencv-python-headless>=4.8.0
pytesseract>=0.3.10
Pillow>=10.0.0
pillow-heif>=0.13.0
groq>=0.4.0
requests>=2.31.0
python-dotenv>=1.0.0
pytest>=7.4.0
```
System dependency: `brew install tesseract` (macOS) / `apt install tesseract-ocr` (Ubuntu) — only needed for the OCR fallback path; the primary vision-based label reader has no system dependencies.

---

### Phase 1 — Project Setup `March 31 – April 1`
> Goal: Get a running Streamlit skeleton with all dependencies installed.

- [x] **1.1 Scaffolding**
  - [x] 1.1.1 Create directory structure (all folders and `__init__.py` files)
  - [x] 1.1.2 Create `requirements.txt` with all packages listed above
  - [x] 1.1.3 Create `.gitignore` (include `.env`, `__pycache__/`, `.venv/`, `*.pyc`)
  - [x] 1.1.4 Create `.env.example` with `GROQ_API_KEY=` and `USDA_API_KEY=`
  - [x] 1.1.5 Set up virtual environment and install dependencies
  - [x] 1.1.6 Install Tesseract OCR system dependency

- [x] **1.2 Minimal Streamlit App**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AaravChadha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
