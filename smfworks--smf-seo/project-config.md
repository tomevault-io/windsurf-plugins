---
trigger: always_on
description: This skill works with:
---

# SMF SEO+GEO — Content Optimization Engine

Create content that ranks in both traditional search AND AI-powered answers. This skill guides you through the complete SEO+GEO content workflow — from keyword research to published draft — with data-driven decisions at every step.

## What It Does

- **Keyword Research**: Find low-competition, high-intent keywords
- **SERP Analysis**: Understand what Google wants for your target keyword
- **Content Outlines**: Structure articles that satisfy search intent
- **SEO-Optimized Drafts**: Write with proper heading hierarchy, internal links, and meta tags
- **GEO Optimization**: Get content cited in ChatGPT, Perplexity, Gemini, and Google AI Overviews
- **Content Refresh**: Update old posts to regain lost rankings in both traditional and AI search

---

## First-Time Setup

**On first use, this skill will launch an interactive onboarding wizard.** The AI will ask you 8 questions to customize the skill for your organization. This takes about 2-3 minutes.

**Questions will cover:**
1. Your website domain
2. Primary services/products
3. Content pillars (main topic areas)
4. Target audience description
5. Key competitors to monitor
6. Existing high-performing content (for internal linking)
7. Brand voice preferences
8. Your name/credentials (for author bylines)

**Your answers are saved locally** in `~/.smf/skills/smf-seo-gee/config.json` — never sent anywhere.

**To re-run setup:** Say "reconfigure SMF SEO" or run `smfw reconfigure smf-seo-gee`

---

## Onboarding Wizard Dialogue

When the user first invokes this skill, conduct this exact dialogue to configure it. Ask one question at a time. After all questions, save the config and confirm.

**Wizard Script:**

```
Welcome to SMF SEO+GEO! I'll configure this skill for your organization.

First, I need to ask you 8 questions. Your answers are saved locally and never sent anywhere.

Let's start:

1. What is your website domain? (e.g., example.com)
   → Store in config.organization.domain

2. What is your organization/company name?
   → Store in config.organization.name

3. What are your primary services or products? (Tell me what you offer)
   → Store in config.organization.primary_services (newline-separated list)

4. What are your main content topic areas? (Your blog categories or expertise areas)
   → Store in config.organization.content_pillars (newline-separated list)

5. Who is your target audience? (Describe your ideal customer or reader)
   → Store in config.organization.target_audience

6. What are your audience's main pain points? (What problems do they face?)
   → Store in config.organization.audience_pain_points (newline-separated list)

7. Who are your main competitors? (Websites you'd like to outrank)
   → Store in config.organization.key_competitors (newline-separated list)

8. Do you have existing high-performing content? (Pages that rank well already)
   → Store in config.organization.existing_high_performing_content (URLs or titles, newline-separated)

Now brand details:

9. How would you describe your brand voice? (e.g., 'Direct and practical' or 'Friendly and conversational')
   → Store in config.brand.voice

10. What name should appear as the author on articles?
    → Store in config.brand.author_name

11. What credentials or bio should appear with articles?
    → Store in config.brand.author_credentials

12. Default target word count for articles? (Press Enter for 2000)
    → Store in config.preferences.default_word_count (number)

That's it! Saving your configuration...
✅ Setup complete! Your SEO+GEO skill is ready.
```

**After saving, run:**
```bash
python3 ~/.smf/skills/smf-seo-gee/scripts/config_manager.py complete
```

**Re-run trigger:** If user says "reconfigure SEO" or "update SEO settings", restart the wizard from question 1.

---

## Quick Start (After Setup)

### 1. New Article from Keyword

"Create an SEO-optimized article targeting the keyword: '[your keyword]'

Target word count: [number]
Tone: [your brand voice]
Include: [comparison table / FAQ / pricing section / etc.]"

### 2. Content Refresh

"Refresh this old blog post for better SEO + GEO performance:
[paste existing content]

Target keyword: [keyword]
Current issues: [dropped rankings / outdated info / etc.]"

### 3. Content Strategy

"Build a 3-month content calendar for [industry] targeting [audience].
Focus topics: [topic 1], [topic 2], [topic 3]
Goal: [leads / brand awareness / authority]"

---

## The SEO+GEO Content Workflow

### Phase 1: Keyword Intelligence (Agent Actions)

**Input:** Seed topic or competitor URL
**Output:** Prioritized keyword list with difficulty scores

**Agent Steps:**
1. Run `web_search` with seed topic to find related queries
2. Run `web_search` with "[topic] vs" and "[topic] how to" to find long-tail variants
3. Run `web_fetch` on Google's "People Also Ask" results
4. Score keywords by: intent match × likely volume × competition signal
5. Output ranked list with reasoning for top 5 picks

**Key metrics to consider:**

- **Search Volume:** Monthly searches (higher ≠ always better)
- **Keyword Difficulty:** Competition level (start with <30 for new sites)
- **Search Intent:** Informational, navigational, transactional, commercial
- **CPC:** Cost-per-click indicates commercial value
- **Trend:** Seasonal or growing interest?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smfworks/SMF-SEO](https://github.com/smfworks/SMF-SEO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
