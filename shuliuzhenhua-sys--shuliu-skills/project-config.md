---
trigger: always_on
description: This repository is a Claude Code skills marketplace currently focused on:
---

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Claude Code skills marketplace currently focused on:
- `banana-proxy`: Gemini image generation via Banana proxy.
- `geek-image`: GeekAI image generation via geekai.co.
- `ecommerce-images`: Workflow skill to generate ecommerce product main/detail images by orchestrating existing image generation skills.
- `sora-video`: Sora video generation via lnapi.com.
- `douyin-share-info`: Fetch Douyin basic info from share URLs via TikHub Web API.
- `wechat-mp-scraper`: Scrape public WeChat article pages, export HTML/content/assets, and analyze animation clues.
- `feishu-user-auth`: Feishu user OAuth/device-flow authorization, scope top-up, and token reuse.
- `feishu-bitable`: Feishu Bitable operations for records, fields, views, permissions, formulas, and links.
- `feishu-approval`: Feishu native approval documentation skill covering approval definitions, form controls, external options, approval instances, and troubleshooting.
- `feishu-card`: Feishu interactive card documentation skill covering card JSON structure, sending, callbacks, and updates.
- `xhs-text2image`: Xiaohongshu text-to-image automation skill for logged-in creator sessions, theme switching, and bundled theme preview assets.

- `.claude-plugin/marketplace.json`: marketplace metadata, plugin groups, and skill registration.
- `skills/banana-proxy/SKILL.md`: user-facing skill contract and usage docs.
- `skills/banana-proxy/scripts/main.ts`: CLI entrypoint for prompt parsing, env loading, single/batch generation.
- `skills/banana-proxy/scripts/providers/google.ts`: Banana proxy Gemini provider implementation.
- `skills/banana-proxy/scripts/types.ts`: shared TypeScript types.
- `skills/geek-image/SKILL.md`: user-facing skill contract and usage docs.
- `skills/geek-image/scripts/main.ts`: CLI entrypoint for GeekAI image generation.
- `skills/geek-image/scripts/providers/geekai.ts`: GeekAI provider implementation.
- `skills/geek-image/scripts/types.ts`: shared TypeScript types.
- `skills/ecommerce-images/SKILL.md`: user-facing skill contract and usage docs for ecommerce images.
- `skills/sora-video/SKILL.md`: user-facing skill contract.
- `skills/sora-video/scripts/main.ts`: CLI entrypoint.
- `skills/sora-video/scripts/providers/lnapi.ts`: Lnapi.com provider implementation.
- `skills/douyin-share-info/SKILL.md`: user-facing skill contract and extraction rules for Douyin share parsing.
- `skills/douyin-share-info/scripts/main.ts`: CLI entrypoint for TikHub API calls and normalized output.
- `skills/wechat-mp-scraper/SKILL.md`: user-facing skill contract for WeChat public-account article scraping.
- `skills/wechat-mp-scraper/scripts/scrape_wechat_mp.py`: Python CLI entrypoint for HTML/content/resource extraction.
- `skills/wechat-mp-scraper/references/output-format.md`: output field reference for generated report/content/resource files.
- `skills/feishu-user-auth/SKILL.md`: user-facing skill contract and Feishu auth workflow.
- `skills/feishu-user-auth/scripts/run-auth.js`: CLI entrypoint for auth/system-token/refresh/show/remove flows.
- `skills/feishu-user-auth/scripts/src/*.js`: OAuth, token store, and scope resolution implementation.
- `skills/feishu-user-auth/config.json`: local skill config template (`appId`, `appSecret`, `brand`).
- `skills/feishu-bitable/SKILL.md`: user-facing skill contract for Feishu Bitable workflows.
- `skills/feishu-bitable/references/fields.md`: field type and `property` reference guide for Bitable field operations.
- `skills/feishu-approval/SKILL.md`: user-facing skill contract for Feishu native approval workflows.
- `skills/feishu-approval/references/*.md`: approval definition, form control, external option, instance, and troubleshooting references.
- `skills/feishu-card/SKILL.md`: user-facing skill contract for Feishu interactive card workflows.
- `skills/feishu-card/references/*.json`: reusable card templates for alert, briefing, and skill-test messages.
- `skills/xhs-text2image/SKILL.md`: user-facing skill contract for Xiaohongshu text-to-image generation, theme updates, and preview catalog routing.
- `skills/xhs-text2image/scripts/xhs_text2image.py`: Python CLI entrypoint for create/update/download/status/themes/catalog flows.
- `skills/xhs-text2image/theme_catalog/`: bundled overview image, manifest, and per-theme sample images for fast customer previews.
- `README.md` / `README.zh.md`: install and update instructions.
- `CHANGELOG.md` / `CHANGELOG.zh.md`: release notes.

## Build, Test, and Development Commands
No build step is required; scripts run directly with Bun.

- Install skill from GitHub:
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill banana-proxy`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill geek-image`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill ecommerce-images`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill sora-video`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill douyin-share-info`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill wechat-mp-scraper`
  - `npx skills add https://github.com/twodogegg/shuliu-skills --skill feishu-user-auth`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuliuzhenhua-sys/shuliu-skills](https://github.com/shuliuzhenhua-sys/shuliu-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
