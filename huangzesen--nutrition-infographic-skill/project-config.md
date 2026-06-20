---
trigger: always_on
description: Generate nutrition/science infographic images through a consultative workflow: first ask the human what image they need (purpose, audience, topic, language, size, style, data/text, output channel), then generate via available backends such as MiniMax CLI, Codex built-in imagegen, OpenAI Images API, or deterministic Pillow fallback.
---


# Nutrition Infographic Generator

This skill creates nutrition/science infographic images. The **default workflow is consultative**: ask the human what kind of image they need before generating anything, unless they already supplied enough detail or explicitly asked for a demo.

Recommended generation route: use whichever image backend is actually available in the current environment. MiniMax CLI (`mmx image generate`) is a good first-party CLI option when a MiniMax key/quota is available; Codex CLI / Codex's built-in `imagegen` system skill is a good route in Codex-enabled environments; the deterministic Pillow renderer remains the fallback for exact chart geometry, repeatability, offline rendering, or fast drafts.

## Self-contained usage

This repository is self-contained. When cloned, run commands from the repository root (the directory containing `SKILL.md`). Do not assume the skill lives at `.library/custom/nutrition-infographic`; examples below use relative paths such as `scripts/render_nutrition_infographic.py` and `assets/demo-balanced-plate.json`. If you copy this skill into LingTai's `.library/custom/` or `.library_shared/`, the same relative paths still work when your shell is inside the skill directory.

### Installing on a new LingTai machine

On a fresh computer or in a new LingTai project, clone the repo and copy or symlink it into the agent's skill catalog:

```bash
git clone https://github.com/huangzesen/nutrition-infographic-skill.git
mkdir -p .library/custom
cp -R nutrition-infographic-skill .library/custom/nutrition-infographic
# or: ln -s "$PWD/nutrition-infographic-skill" .library/custom/nutrition-infographic
```

Then refresh the agent so the skill catalog is rescanned. After refresh, the agent only needs to read this `SKILL.md` to know the workflow. The chat history from the machine where the skill was authored is not required.

Prerequisites for polished AI-image routes: at least one image backend must be installed/authenticated and have quota, such as MiniMax CLI (`mmx`) or command-line `codex` with the system imagegen skill / built-in `image_gen` capability. If no AI-image backend is available, use the OpenAI API fallback when credentials exist, or the deterministic Pillow fallback below.

## When this applies

Use this skill for:

- balanced-plate diagrams and meal composition visuals
- macronutrient proportion charts
- nutrition education cards, posters, and article illustrations
- diet/health science explainers where numbers and labels must stay legible
- rapid drafts that should look like finished educational images
- social posts, article covers, slides, handouts, and app onboarding visuals

Do **not** use this as-is for medical diagnosis, patient-specific diet prescriptions, disease-treatment promises, photorealistic food photography, or brand-sensitive publication design without explicit human direction and review.

## First move: ask what image the human needs

If the human has not already provided the essentials, ask a short intake question before generating. Do not interrogate them with a long form; ask for the missing pieces that matter.

### Minimal intake

Ask in the human's language:

> 你想要一张什么样的营养学图片？请告诉我：用途/平台、主题、面向谁、语言、尺寸比例、风格、是否有必须出现的数据或文案。没有细节的话我可以先给你一个默认方案。

For English:

> What nutrition image do you need? Please tell me the purpose/platform, topic, audience, language, aspect ratio, style, and any required data or wording. If you do not have details, I can propose a default.

### If they are in a hurry

Ask only the 3 essentials:

1. **Topic** — what nutrition concept, food, meal, nutrient, or behavior?
2. **Use/channel** — WeChat article, poster, slide, social card, app UI, handout?
3. **Audience/language** — general adults, children, athletes, older adults, diabetes education, etc.; Chinese/English/bilingual?

Then choose sensible defaults for the rest and state them before generating.

### Intake fields to capture

- **Purpose/channel**: WeChat article, service-account reply, Xiaohongshu card, slide, poster, handout, app screen.
- **Audience**: general public, parents, students, athletes, older adults, clinicians, patients, etc.
- **Topic**: balanced meal, protein, fiber, glycemic load, hydration, sodium, pregnancy nutrition, etc.
- **Message goal**: educate, compare, warn, motivate, summarize, explain a process.
- **Language**: Chinese, English, bilingual; simplified/traditional if Chinese matters.
- **Canvas**: square `1024x1024`, vertical `1080x1920`, horizontal slide `1600x900`, print A4, etc.
- **Style**: clean medical, warm editorial, playful cartoon, premium brand, minimalist data viz, etc.
- **Required text/data**: exact title, numbers, units, source, disclaimer.
- **Brand constraints**: colors, logo, QR code, no-logo, typography.
- **Accuracy constraints**: illustrative vs sourced; whether medical/professional review is required.
- **Output path/channel**: where to save and whether to send/attach it.

## Decision tree


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangzesen/nutrition-infographic-skill](https://github.com/huangzesen/nutrition-infographic-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
