---
trigger: always_on
description: Comprehensive fitness and diet tracking with natural language meal logging, workout logging, food image analysis, automatic calorie/macro calculation, daily and weekly health summaries integrating Fitbit data, hydration tracking, saved meal shortcuts, exercise standardization (60+ exercises), progressive overload tracking with PR history, workout templates, recovery monitoring, natural language workout queries, and adaptive training suggestions. Use for logging meals ("I ate X for lunch") or wor
---


# Fitness & Diet Management

## Coaching Philosophy

1. **Consistency over perfection** -- A logged imperfect day is more valuable than an untracked perfect one. Encourage logging even when choices aren't ideal.
2. **Progress the weakest link** -- Identify the one area that will yield the most improvement and focus coaching notes there rather than listing everything at once.
3. **Recovery is training** -- Sleep, rest days, and stress management are as important as workouts. Flag poor recovery patterns proactively.
4. **Small sustainable changes > dramatic overhauls** -- Recommend incremental adjustments (e.g., add one serving of protein per day) rather than complete diet rewrites.
5. **Data informs, doesn't dictate** -- Use trends and averages for coaching, not single-day outliers. One bad day doesn't erase a good week.
6. **Celebrate wins, don't just flag problems** -- Lead with strengths in coach's notes before areas for improvement. Positive reinforcement builds habit adherence.

## User Goals

All thresholds and targets are configurable via `config.json` under the `GOALS` section. When weight/height/age are set, calorie targets are calculated automatically using the Mifflin-St Jeor equation with activity multipliers.

| Setting | Default | Description |
|---------|---------|-------------|
| `goal_type` | maintenance | One of: maintenance, weight_loss, muscle_gain |
| `weight_kg` | null | Body weight in kg (enables protein target calculation) |
| `height_cm` | null | Height in cm (enables calorie target calculation) |
| `age` | null | Age in years (enables calorie target calculation) |
| `sex` | male | male or female (affects BMR calculation) |
| `activity_level` | moderate | sedentary, light, moderate, active, very_active |
| `protein_per_kg` | 0.8 | Protein target multiplier per kg body weight |
| `calorie_target` | null | Manual override for daily calorie target |
| `sodium_limit_mg` | 2300 | Daily sodium limit in mg |
| `fiber_target_g` | 38 | Daily fiber target in grams |
| `step_target` | 10000 | Daily step target |
| `sleep_target_h` | 7.0 | Daily sleep target in hours |

## Dietary Profile

Personalization for allergies, dietary restrictions, health conditions, and food preferences. Configured via `config.json` under the `DIETARY_PROFILE` section.

| Setting | Default | Description |
|---------|---------|-------------|
| `allergies` | [] | Food allergies (e.g., peanuts, shellfish, dairy, gluten) |
| `dietary_restrictions` | [] | Dietary restrictions (e.g., vegetarian, vegan, keto) |
| `dislikes` | [] | Foods to avoid in meal suggestions |
| `cuisine_preferences` | [] | Preferred cuisines (e.g., italian, mexican, asian) |
| `health_conditions` | [] | Health conditions affecting diet (e.g., diabetes, hypertension) |
| `cooking_skill` | null | Cooking skill level: basic, intermediate, advanced |
| `budget` | null | Food budget: budget, moderate, premium |
| `meal_timing` | null | Typical meal schedule |
| `meal_variety` | "balanced" | Recommendation style: explore, balanced, or consistent |
| `notes` | "" | Additional dietary notes |

**Gradual learning:** Preferences are learned over time through coach note prompts. Safety-critical items (allergies, health conditions) are asked after the first meal log. Other preferences are prompted gradually:
- Allergies + health conditions: after 1st meal (safety)
- Dietary restrictions: after 5th meal (coaching)
- Dislikes: after 10th meal (comfort)
- Cuisine preferences: after 15th meal or 1st meal plan request (planning)
- Cooking skill / budget: on 1st meal plan request (planning)
- Meal timing: after 20th meal (optimization)

**Quick setup:** Say "let's configure" to set all preferences at once.

**CLI:**
- `python3 scripts/dietary_profile.py --show` — View current profile
- `python3 scripts/dietary_profile.py --set allergies "peanuts,shellfish"` — Set a preference
- `python3 scripts/dietary_profile.py --next-prompt` — Get next gradual learning prompt
- `python3 scripts/dietary_profile.py --full-setup` — Get all unset preference prompts

## Core Capabilities

This skill manages personal health data through eight integrated workflows:

### 1. Natural Language Meal Logging
Process natural language food descriptions and automatically log to daily diet file with calorie/macro estimates using multi-source food database (750,000+ foods from local SQLite, OpenNutrition, and USDA API). Beverages are automatically tracked for hydration. Saved meal shortcuts are expanded before parsing.

**Triggers:**
- "I ate/had [food] for [meal]"
- "Just finished [food] for [meal]"
- "For lunch today I had [food]"

**Process:**
1. Expand saved meal shortcuts (see Saved Meals below)
2. Parse meal type (breakfast, lunch, dinner, snack)
3. Extract food items and quantities
4. Look up in food database (see references/food-database.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickulrich/health-skill](https://github.com/patrickulrich/health-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
