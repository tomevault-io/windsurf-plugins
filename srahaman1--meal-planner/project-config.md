---
trigger: always_on
description: This file provides guidance to Claude Code and other AI assistants when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code and other AI assistants when working with this repository.

## Project Overview

**Meal Planner** is an automated meal planning system for households with individualized macro targets and diabetic-friendly recipes.

**Key Features:**
- Automated recipe selection with 2-week cooldown
- Allergen filtering (hard filter - removes recipes)
- Dislike flagging (soft filter - warns but includes recipes)
- Portion calculation based on individual macro targets
- Grocery list generation
- Interactive CLI interface

**Target Users:**
- Households managing diabetes or pre-diabetes
- Families with different macro targets per person
- Batch meal prep on Sundays
- 3 meals/day with rotating recipes

## Architecture

### Core Principles

**YAGNI (You Aren't Gonna Need It):**
- Standard library only (no external dependencies)
- Simple algorithms (greedy selection, proportional allocation)
- POC-focused implementation
- Complex optimizations deferred to future

**DRY (Don't Repeat Yourself):**
- Modular design (selector, calculator, generator)
- Reusable utilities (JSON I/O, date helpers)
- Single source of truth for data (JSON files)

### Tech Stack

- **Python 3.8+** (standard library only)
- **JSON** for data storage (human-readable, easy to edit)
- **Markdown** for recipes (Obsidian-compatible)

### Key Design Decisions

**Why no external dependencies?**
- Faster iteration (no dependency management)
- Easier deployment (just Python + repo)
- Lower barrier to contribution
- Sufficient for POC scope

**Why JSON instead of database?**
- Human-readable and editable
- Version control friendly
- No setup required
- Sufficient for 265 recipes + 3 personas

**Why greedy algorithm instead of optimization?**
- Fast and "good enough" for POC
- Complexity doesn't justify optimization yet
- Can upgrade to constraint solver later if needed

## Project Structure

```
meal-planner/
├── meal_planner/              # Main package
│   ├── cli.py                 # Interactive CLI entry point
│   ├── utils.py               # JSON I/O, date helpers
│   ├── recipe_loader.py       # Allergen/dislike filtering
│   ├── recipe_selector.py     # Selection algorithm
│   ├── portion_calculator.py  # Serving size math
│   ├── plan_generator.py      # Complete plan generation
│   ├── grocery_generator.py   # Grocery list generation
│   └── swap_engine.py         # Find similar recipes
├── data/
│   ├── recipes/               # 265 markdown recipe files
│   ├── recipe-metadata.json   # Complete recipe database
│   ├── macro-profiles.json    # Macro target templates
│   └── personas.json          # Individual household members
├── scripts/                   # Supporting utilities
└── docs/                      # Documentation
```

## Data Model

### Macro Profiles

**Purpose:** Reusable macro target templates

**Structure:**
```json
{
  "id": "weight-loss-high-protein",
  "name": "Weight Loss - High Protein",
  "calories": 1600,
  "protein": 150,
  "carbs": 110,
  "fat": 62,
  "notes": "High protein for satiety"
}
```

**Key Insight:** Macro profiles are templates that personas reference. Multiple personas can share the same profile.

### Personas

**Purpose:** Individual household members with allergies and preferences

**Structure:**
```json
{
  "id": "john",
  "name": "John",
  "macro_profile": "weight-loss-high-protein",
  "allergies": ["peanuts", "shellfish"],
  "dislikes": ["raw-tomatoes"],
  "dietary_restrictions": [],
  "notes": "Pre-diabetic, weight loss goal"
}
```

**Key Distinctions:**
- **Allergies:** Hard filter - recipes completely removed from selection
- **Dislikes:** Soft filter - recipes flagged but still selectable (user can swap)
- **Dietary restrictions:** Not yet implemented

### Recipes

**Metadata:**
```json
{
  "slug": "korean-bbq-chicken-rice-bowls",
  "name": "Korean BBQ Chicken Rice Bowls",
  "category": "meals",
  "calories_per_serving": "580",
  "protein": "52g",
  "carbs": "48g",
  "fat": "22g",
  "key_ingredients": "chicken breast, rice, gochujang"
}
```

**Categories:**
- `breakfast` - Morning meals (26 recipes)
- `meals` - Lunch/dinner recipes (198 recipes)
- `sides` - Side dishes (25 recipes)
- `snacks` - High-protein snacks (16 recipes)

**Source:** Extracted from two high-protein meal prep cookbooks (265 total recipes)

### Meal Plans

**Structure:**
```json
{
  "plan_id": "2026-02-10",
  "week_start": "2026-02-10",
  "week_end": "2026-02-23",
  "recipe_summary": [...],
  "schedule": [...]
}
```

**recipe_summary:** Lists each recipe with total servings needed and breakdown by persona

**schedule:** 14-day daily schedule with portion sizes per persona per meal

## Algorithms

### Recipe Selection

**Algorithm:** Greedy selection with filters

**Process:**
1. Load all 265 recipes
2. **Hard filter:** Remove recipes with allergens (checks `key_ingredients`)
3. **Cooldown filter:** Remove recipes used in last 2 weeks
4. **Soft filter:** Flag recipes with disliked ingredients
5. Split by category (breakfast vs meals)
6. Select first N available (3 breakfasts, 4 meals)
7. Build 14-day schedule (rotate recipes)

**Improvement opportunities:**
- Optimize for macro targets (currently just picks first N)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srahaman1/meal-planner](https://github.com/srahaman1/meal-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
