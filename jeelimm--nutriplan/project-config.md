---
trigger: always_on
description: **NutriPlan** is an AI-powered calorie and meal management app.
---

# NutriPlan — CLAUDE.md

## Project Overview

**NutriPlan** is an AI-powered calorie and meal management app.
Users enter their body metrics and a goal (fat loss, muscle gain, or recomposition),
and the Anthropic Claude API automatically generates a personalized 7-day meal plan and grocery list.

- Unidirectional flow: Onboarding → Meal Plan Config → Daily View → Grocery List
- Two onboarding paths: **Quick Estimate** (default — body type → estimated body composition) and **InBody / Precision** (exact measurements)
- Supports Korean and English, and multiple cuisine styles (Korean, Japanese, Western, etc.)
- Designed and deployed on Vercel

Refer to UX_FLOW.md in the project root for the complete screen inventory, user flows, navigation map, and known pain points before making any UI changes.

---

## Tech Stack

| Category | Technology |
|----------|------------|
| Framework | Next.js 16.2 (App Router) |
| UI Library | React 19 + TypeScript 5.7 |
| Styling | Tailwind CSS v4 + shadcn/ui (Radix UI based) |
| Icons | Lucide React |
| Global State | Zustand v5 (with persist middleware) |
| AI | Anthropic SDK (`claude-haiku-4-5-20251001`) |
| Charts | Recharts |
| Forms | React Hook Form + Zod |
| Notifications | Sonner (toast) |
| Deployment | Vercel (with `@vercel/analytics`) |

---

## File Structure

```
app/
├── page.tsx                        # Root entry point — controls screen switching via currentStep
├── layout.tsx                      # Global layout (fonts, metadata)
├── globals.css                     # Global CSS (Tailwind layers)
└── api/
    ├── generate-meal-plan/
    │   └── route.ts                # Generates 7-day meal plan via Claude
    ├── swap-meal/
    │   └── route.ts                # Generates 3 swap candidates for a single meal via Claude
    └── claude-nutrition/
        └── route.ts                # Looks up nutrition data for a custom ingredient via Claude

components/
├── onboarding.tsx                  # Multi-step onboarding wizard (Step 0) — two entry paths
├── meal-plan-config.tsx            # Meal plan configuration screen (Step 1)
├── daily-view.tsx                  # Daily meal view with swap, edit-profile modal (Step 2)
├── grocery-list.tsx                # Full-week grocery list screen (Step 3)
├── grocery-item-row.tsx            # Individual grocery item row component
├── meal-swap-sheet.tsx             # Bottom sheet overlay for meal swapping (used inside DailyView)
├── settings-screen.tsx             # App preferences + profile quick-edit (Step 4)
├── theme-provider.tsx              # Dark mode theme provider
└── ui/                             # shadcn/ui components (can be modified directly)

lib/
├── meal-store.ts                   # Zustand global state + core type definitions
├── nutrition.ts                    # Calorie and macro target calculation (Katch-McArdle)
├── meal-validator.ts               # Validates generated meal plans and swap candidates
├── grocery.ts                      # Grocery list utilities
├── recipe-units.ts                 # Unit conversion (metric / imperial)
└── utils.ts                        # Shared utilities (clsx + tailwind-merge)

hooks/
├── use-mobile.ts                   # Mobile detection hook
└── use-toast.ts                    # Toast notification hook

.github/workflows/
├── ci.yml                          # Build verification on push/PR to main
└── ai-review.yml                   # AI-powered PR review via Claude Code Action

.claude/commands/
├── prd.md                          # /project:prd slash command
├── implement.md                    # /project:implement slash command
└── pipeline.md                     # /pipeline full automation command (PRD → prompts.json → execute)

prompts.json                        # Current feature prompts consumed by run-prompts.sh
run-prompts.sh                      # Automation script: runs prompts → build check → commit
```

### App Flow (by currentStep)

```
0: Onboarding → 1: MealPlanConfig → 2: DailyView → 3: GroceryList
                                                  ↕ (FAB button)
                                              4: SettingsScreen
```

---

## Onboarding Flow

The onboarding wizard (`components/onboarding.tsx`) collects all data needed to compute calorie and macro targets. It has two entry paths that converge at the same steps from "activity level" onward.

### Path 1 — Quick Estimate (default)

The default path. Estimates body fat % and muscle mass from visible body type so users without lab results can still get accurate targets.

1. User enters: **sex**, **weight**, **height**, **age**, **body type** (lean / average / athletic / heavy-set).
2. Taps "Use quick estimate" → `applyQuickEstimate()` derives `bodyFat %` and `muscleMass` from a lookup table (`quickEstimateMap[sex][bodyType]`).
3. Continues to: activity level → goal → target weight (conditional) → cuisine → diet type → ingredient mode → ingredients → "Build my plan".

### Path 2 — InBody / Precision

For users with exact body composition measurements (e.g., InBody scanner).

1. User taps "Enter exact body stats" → body stats form: **weight**, **body fat %**, **muscle mass**, **sex**, **unit system**.
2. Steps from activity level onward are identical to Quick Estimate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeelimm/nutriplan](https://github.com/jeelimm/nutriplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
