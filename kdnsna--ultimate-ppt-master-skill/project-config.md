---
trigger: always_on
description: >
---


# 终极融合PPT大师 Skill

> Ultimate Fusion PPT Master: a Codex skill that fuses editable PPTX generation with magazine-style web deck generation.

**Core Pipeline**: `Request → Source Document → One Delivery Brief → Assets → Generate → Verify → Export/Preview`

**Quality Pipeline Add-on**: stakeholder-facing decks must also pass `Visual Direction → Page Role Contract → Visual Completion Audit`.

## Default Delivery Route

The skill is optimized for real office PPT delivery. Keep the route decision simple:

- For generic requests such as "做一个 PPT", "帮我做 PPT", "make a deck", or "turn this into slides", default to **Mode 1: Editable PPTX** and proceed. Say briefly that Web Deck remains available if the user wants a showcase-style browser version.
- Choose **Mode 1: Editable PPTX** when the user asks for PowerPoint, `.pptx`, editable files, business reports, consulting decks, training material, government/finance material, or anything another person may need to revise.
- Choose **Mode 2: Magazine Web Deck** only when the user explicitly asks for magazine style, web PPT, HTML slides, horizontal swipe deck, editorial/e-ink, Swiss Style, keynote/showcase/demo-day style, or a browser-first deliverable.
- If the user asks for both, produce separate deliverables in separate project folders.
- Ask the user to choose only when the request is genuinely route-ambiguous and the scenario cannot decide it. Ask one concise question, then continue.

Plain-language wording for generic requests:

> 我会默认做可编辑 PPTX，适合正式汇报和后续修改；如果你想要杂志风网页 PPT，我也可以改走 HTML 版本。

## Formal Business Delivery Gate

Default to `qualityGate.level = "formal-business"` for business/report/consulting/training/government/finance decks and for any deliverable expected to be handed to a real stakeholder. This is the default quality bar unless the user explicitly asks for a quick draft.

Before generating final PPTX or Web Deck files, lock these items in `design_spec.md`, `spec_lock.md`, and `design-quality-report.md`:
- Visual direction selection from `templates/visual-directions/`, or a documented `custom` benchmark when no direction fits.
- Page role and recipe contract: every page has `page_role`, `visual_weight`, `layout_family`, `page_recipe_id`, `asset_requirement`, `visual_layer`, `raster_policy`, and `anti_patterns` in `design_spec.md` / `spec_lock.md`.
- Brand assets or a documented fallback strategy.
- Official/IP asset plan for deterministic marks mentioned by the source or user, such as company logos, campaign logos, tourism IP, product marks, QR codes, seals, and partner marks. Search official or authorized sources first; never draw a fake logo-shaped placeholder.
- Traceable evidence sources and data interpretation boundaries.
- Codex native GPT image generation (`image2` when available) as the default visual asset engine for custom visuals; this is the Codex execution path for "ChatGPT/OpenAI as the primary visual asset engine". Record prompts, filenames, target slides, and manual edits in `asset-plan.md` or `images/image_prompts.md`.
- Reusable small element asset plan when useful: section dividers, metric badges, process nodes, connectors, icon accents, textures, and callout stickers.
- Local element generation state from `scripts/generate_visual_element_kit.py`: `assets/generated/element-manifest.json`, `images/image_prompts.json`, and `images/image_prompts.md`.
- Public asset search plan for evidence, official references, and brand boundaries, or explicit no-search rationale; record source URL, publisher, license/usage note, and insertion target for each selected public asset.
- `spec_lock.md brand_assets` and `asset-plan.md` / `images/image_sources.json` entries for every deterministic IP mark that will appear in the deck. Each entry must record `official-source`, `user-provided`, `text-lockup-fallback`, or `needs-authorized-replacement`.
- Image, chart, and infographic plan, or an explicit no-image strategy.
- Page rhythm, layout variety, and the role of each slide.
- `spec_lock.md aesthetic_checks` covering body font baseline, title/body scale, card count, card padding, whitespace target, logo handling, and repeated-layout risk.
- 4.0 hybrid-editable visual strategy: generated visuals support the page as no-text layers; formal body pages remain editable.
- Visual completion status: screenshots/PNGs rendered, repeated-layout risk checked, placeholder assets labeled, and design-quality-report written.
- Artifact checks for editable PPTX objects and complete Web Deck visual rendering.

Formal delivery rules:
- Do not build a whole deck from repeated title-and-card pages.
- Do not let three consecutive content pages use the same layout family unless the Design Spec records a deliberate reason.
- Do not let three consecutive content pages use the same page recipe unless the Design Spec records a deliberate reason.
- Do not use full-page generated images for formal PPTX body pages. Full-page raster is allowed only for covers, section/tail pages, poster/KV pages, Web showcase pages, or explicit user override recorded in `raster_policy`.
- Do not proceed with only slide titles unless the deck is clearly marked as a draft and the user accepts that limitation.
- Do not use a generic "free design" look when a visual direction pack matches the deck context. Select the direction pack first, then adapt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdnsna/ultimate-ppt-master-skill](https://github.com/kdnsna/ultimate-ppt-master-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
