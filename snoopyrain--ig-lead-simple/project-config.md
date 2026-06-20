---
trigger: always_on
description: >
---


# IG Lead Magnet Simple (v1)

> **Public release notice**: this skill was extracted from a personal toolkit. All account IDs, MCP URLs, GCS bucket names, vault paths, and product names have been replaced with `<PLACEHOLDER>` tokens. See [README.md](README.md) for setup; see [config.example.md](config.example.md) for the full placeholder list.

The fastest IG lead-magnet loop: comment keyword → public reply → DM hands a Google Drive link directly. **No email capture, no nurture sequence**. You trade the CRM list for the lowest-friction validation speed.

## Usage

```
/100m-leads:ig-simple <product> [language]
```

| Arg | Values | Default |
|---|---|---|
| `product` | One of the products listed in your local config (see `config.example.md`). | required |
| `language` | `zh` / `zh-Hant` (**Traditional Chinese**) or `en` (English) | `zh` |

> 🔴 **Language hard rule**: `zh` = **Traditional Chinese (zh-Hant)**, **simplified Chinese is forbidden**. Every prompt, PDF body, caption, DM, and carousel text overlay must explicitly say `Traditional Chinese` / `繁體中文` so the LLM / gpt-image-2 doesn't render simplified.

**Examples** (the actual handles come from your `config.example.md`):

- `/100m-leads:ig-simple <your-product-A> zh` → Traditional-Chinese run → IG `<your_zh_handle>` + TikTok `<your_zh_tiktok_name>`
- `/100m-leads:ig-simple <your-product-B> en` → English run → IG `<your_en_handle>` + TikTok `<your_en_tiktok_name>`

The skill will automatically:
- Read the product's PRD / `lead-magnet.md` / `design-guideline.md` / `campaign/*.md` / image lists
- Apply the product-specific design system (color preset comes from `design-guideline.md`, not the YAML default)
- Route to the language-matched IG + TikTok account (**dual-platform synchronized publish**)
- Run all 5 phases; pause for final confirmation right before Phase 4 publishes

> The point of skipping interactive questions: stop re-asking "which product / which palette / which account" every run. Decisions are 90% automatic; you only pick the lead-magnet topic and hit publish.

```
        ┌─────────────────────────────────────────┐
        │            parallel execution           │
        │                                         │
        │  ┌──────────────┐  ┌──────────────┐     │
        │  │ Phase 1      │  │ Phase 2      │     │
        │  │ Lead Magnet  │  │ IG Carousel  │     │
        │  │ PDF + Drive  │  │ (gpt-image-2)│     │
        │  └──────────────┘  └──────────────┘     │
        │                                         │
        │  ┌──────────────┐                       │
        │  │ Phase 3      │                       │
        │  │ Caption + DM │                       │
        │  └──────────────┘                       │
        └────────────────────┬────────────────────┘
                             ▼
                  ┌──────────────┐    ┌───────────┐
                  │ Phase 4      │    │ Phase 5   │
                  │ Boring publ. │ →  │ Metrics   │
                  │ +autoReply+DM│    │           │
                  └──────────────┘    └───────────┘
```

> **When to use this vs v2 (`ig-leadmagnet-machine`)**: you want to validate a hook / lead-magnet's market fit fast, you don't want to build an email nurture, the product is too early to justify a list. If you need a nurture sequence, switch to v2.

## References

| File | Purpose |
|------|---------|
| [`references/google-drive-upload.md`](references/google-drive-upload.md) | Drive API flow, folder structure, helper script, permissions |
| [`references/boring-mcp.md`](references/boring-mcp.md) | Boring publish + auto-reply + DM Quick Reply |
| [`references/copywriting-skills.md`](references/copywriting-skills.md) | Caption / Carousel / DM structure (Sugarman framework) |
| [`references/media-creation.md`](references/media-creation.md) | VPick gpt-image-2 prompt patterns, 7-page immersive carousel design |
| [`references/carousel-design-system.yaml`](references/carousel-design-system.yaml) | **Authoritative design source** — 7-page narrative, 4 palette presets, type / grid / cross-page |

---

## Phase 0: Pre-flight + product confirmation

### 🔴 Step 0.1: MCP connection health check

```bash
claude mcp list | grep -E "boring|vpick|obsidian"
```

**All three must be `✓ Connected`**:

| MCP | URL | Purpose |
|-----|-----|---------|
| `boring` | `<your-boring-host>/mcp/t/<token>` | IG publish + auto-reply + DM |
| `vpick` | `<your-vpick-host>/mcp/t/<token>` | Carousel images (gpt-image-2 @ 1K @ 4:5) |
| `obsidian-mcp` | `<your-obsidian-mcp-host>/mcp` | Read/write Obsidian campaign records (cross-machine) |

If any shows `✗ Failed to connect`:

```bash
claude mcp remove <name>
claude mcp add --transport http <name> <url>
# /exit and restart the session — `claude mcp add` does not load deferred tools live
```

> First time on a new machine: run all three `claude mcp add` commands, then `/exit` and restart.

### 🔴 Step 0.1.5: Sandbox permission pre-check (**required**)

Auto-mode harness blocks "Create Public Surface" by default (publishing to IG, uploading public Drive files, calling public APIs). Phase 1 + Phase 4 hit these. **Before running the skill, confirm `.claude/settings.local.json` has these 7 allow rules**; surface what's missing for the user to copy-paste, then proceed to Phase 1.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snoopyrain/ig-lead-simple](https://github.com/snoopyrain/ig-lead-simple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
