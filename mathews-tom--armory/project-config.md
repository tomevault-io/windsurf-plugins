---
trigger: always_on
description: Transforms topics and source material into channel-optimized content across
---


# Content Strategist

Transforms topics and source material into channel-optimized content across
multiple formats through a structured research-strategy-production pipeline.

---

## Scope and Trigger Conditions

### Activate when:

- User requests content creation across multiple channels or formats
- User wants to repurpose existing material (article, video, talk) into new formats
- User asks for a content strategy or content plan for a topic
- User needs a LinkedIn post, blog article, slide deck, or PDF report produced
- User provides a topic and wants channel recommendations
- User wants to turn research or technical material into publishable content

### Do NOT activate when:

- User asks for a single LinkedIn post with no broader strategy (use `linkedin-post-style` skill)
- User asks only to convert markdown to PDF (use `md-to-pdf` skill)
- User asks only to build slides (use `html-presentation` skill)
- User asks only to humanize existing text (use `humanize` skill)
- User asks for web research without content production (use `tavily` skill)
- User asks for YouTube video analysis without content production (use `youtube-analysis` skill)

---

## Input Requirements

| Input                    | Required | Description                                                                                   |
| ------------------------ | -------- | --------------------------------------------------------------------------------------------- |
| Topic or source material | Yes      | A topic description, URL, document, video link, or existing content to work from.             |
| Target channels          | No       | Specific formats requested (blog, LinkedIn, slides, PDF). Agent recommends if omitted.        |
| Audience                 | No       | Target audience profile. Defaults to technical professionals.                                 |
| Tone                     | No       | Desired voice (conversational, formal, authoritative). Agent selects per channel if omitted.  |
| Goal                     | No       | Content goal (awareness, education, conversion, thought leadership). Agent infers if omitted. |

---

## Composition Map

| Component           | Type  | Invoked In | Purpose                                        |
| ------------------- | ----- | ---------- | ---------------------------------------------- |
| tavily              | skill | Phase 2    | Topic research and fact gathering              |
| youtube-analysis    | skill | Phase 2    | Extract insights and quotes from video sources |
| linkedin-post-style | skill | Phase 4    | Platform-optimized LinkedIn post generation    |
| html-presentation   | skill | Phase 4    | HTML slide deck production                     |
| md-to-pdf           | skill | Phase 4    | Polished PDF document output                   |
| humanize            | skill | Phase 5    | Remove AI patterns, ensure natural voice       |

---

## Workflow Phases

### Phase 1: Goal & Audience Analysis

1. Parse the user's request to determine:
   - **Content goal:** awareness, education, conversion, or thought leadership
   - **Target audience:** developers, executives, general technical, non-technical
   - **Existing assets:** URLs, documents, videos, or raw notes to repurpose
   - **Desired channels:** explicit requests or gaps to recommend
2. Ask up to 3 clarifying questions if critical information is missing:
   - What is the primary goal of this content?
   - Who is the target audience?
   - Which channels/formats matter most?
3. Do not ask questions if the request provides sufficient context to proceed.

### Phase 2: Research & Source Gathering

1. If the topic requires factual grounding or the user's source material is thin:
   - Invoke the `tavily` skill to research the topic, gather current data, statistics, and expert perspectives
2. If the user provides video URLs or references video content:
   - Invoke the `youtube-analysis` skill to extract key insights, quotes, timestamps, and structural patterns
3. Compile a source brief: key facts, data points, quotes, and narrative angles gathered from research
4. Skip this phase entirely if the user provides comprehensive source material that needs no supplementation.

### Phase 3: Content Strategy

1. Define the **core message** — one sentence that every content piece reinforces
2. Identify **3-5 key points** that support the core message with evidence from Phase 2
3. Map key points to channels, determining which formats serve the goal:
   - **Long-form** (blog/article): deep exploration, SEO value, education goals
   - **Social** (LinkedIn): awareness, thought leadership, conversational reach
   - **Presentations** (slides): education, internal communication, conference talks
   - **Documents** (PDF): formal reports, whitepapers, executive summaries
4. Define channel-specific adaptations:
   - What angle each channel takes on the core message
   - What length and depth each channel requires
   - What CTA (if any) each channel uses
5. Present the content plan to the user before proceeding to production.

### Phase 4: Content Generation

Produce content for each planned format. Order of production: long-form first (establishes depth), then adapt to other channels.

**Long-form (blog/article):**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mathews-Tom/armory](https://github.com/Mathews-Tom/armory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
