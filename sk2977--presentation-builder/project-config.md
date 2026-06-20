---
trigger: always_on
description: Create professional, branded slide presentations from storyboards or from scratch. Produces HTML/CSS decks with data charts, illustrated graphics, callout boxes, and timelines -- exported as PDF. Use this skill whenever the user wants to build a presentation, create slides, make a deck, design a workshop, or turn research/content into visual slides. Also use when the user mentions PowerPoint, PPTX, keynote, slide deck, storyboard-to-slides, or wants to present information visually to an audience
---


# Presentation Builder

Build professional, story-driven slide presentations from storyboards. Produces HTML/CSS decks with branded styling, data visualizations, and structured layouts -- exported as PDF for presenting.

The presentations this skill creates look like they were designed by a professional agency -- not like default PowerPoint templates. They use CSS Grid for precise layouts, matplotlib for publication-quality charts, inline SVG for icons and illustrations, and a consistent brand system throughout.

## When to Use This Skill

- User wants to create a presentation, slide deck, or workshop materials
- User has a storyboard or outline they want turned into slides
- User wants to research a topic AND produce a presentation about it
- User mentions slides, deck, PowerPoint, PDF presentation, keynote, or workshop
- User has content (research, notes, bullet points) they want to present visually

## End-to-End Workflow

The skill operates in two phases. Phase 1 can be skipped if the user already has a storyboard.

### Phase 1: Storyboard Creation (if needed)

If no storyboard exists, create one through research and synthesis.

**Step 1: Understand the presentation context**

Ask the user (skip questions they've already answered):
- What is the topic and who is the audience?
- How long is the presentation? (determines slide count: ~1 slide per 5-7 minutes)
- What is the narrative arc? (chronological, problem-solution, journey, comparison)
- Are there co-presenters? What sections do they own?
- Any specific data, case studies, or examples to include?

**Step 2: Research (if the topic requires it)**

Deploy parallel research agents (up to 5) to gather material:
- Each agent gets a specific subtopic with a clear objective
- Each must return structured summaries with sources/citations
- Synthesize findings after all agents return

**Step 3: Write the storyboard**

Create `storyboard.md` with this structure for each slide:

```markdown
## SLIDE N: [Title]
**Title:** "[Display title]"

### Setting the Scene
[Narrative context -- what's happening at this point in the story]

### Key Content
[Bullet points, data, tables -- the substance of the slide]

### Messaging
> [The key takeaway quote or message for this slide]

### Graphic Concept
[What visual/chart/diagram should appear and what it communicates]

### Audience Question (optional)
> [Discussion prompt to engage the audience]
```

Also include:
- **Workshop flow & timing** table (if it's a workshop/talk)
- **Graphics to produce** inventory (numbered list of every visual asset)
- **Case studies** section (if applicable)

Save as `storyboard.md` in the project directory. Present to user for review before proceeding.

### Phase 2: Visual Production

Turn the storyboard into a finished presentation.

**Step 1: Establish the style system**

Check if a `style_guide.md` exists. If not, ask the user:
- What organization/brand is this for? (to look up official colors)
- Or: what are 2-3 brand colors? (hex codes or color names)
- Serif or sans-serif preference?
- Professional/corporate or modern/startup feel?

If the user gives a brand name, research official brand colors and typography. Then generate `style_guide.md` with:
- Color palette (primary, secondary, accent, neutral, functional)
- Typography scale (heading sizes, body size, caption size -- presentation scale, not document scale)
- Component specs (callout boxes, stat boxes, tables, timelines)
- Chart color assignments (consistent series colors across all charts)
- Print/PDF settings

If a style guide already exists, read it and use it as-is.

**Step 2: Analyze the storyboard for visual assets**

Read the storyboard and catalog every graphic needed:
- **Data charts** (pie, bar, line, dual-axis, stacked) -- will use matplotlib
- **Layout graphics** (timelines, matrices, grids) -- will use HTML/CSS
- **Icons/illustrations** (characters, objects) -- will use inline SVG or icon libraries
- **Tables** -- will render as styled HTML tables

Create a mental inventory. You do not need to write a separate graphic_preparation_plan.md unless the user asks for one -- just proceed to production.

**Step 2b: Generate illustrations with Recraft V3 (optional)**

If the user wants illustrated graphics (characters, objects, icons) rather than plain inline SVG, use the Recraft V3 API to generate professional vector illustrations. This produces significantly higher-quality visuals than Claude-generated SVG -- the difference is like clip art vs. professional illustration.

**When to use Recraft:**
- User explicitly asks for illustrations, icons, or visual storytelling elements
- The presentation has characters/roles that need visual representation (e.g., "scientist", "investor", "patient")
- The deck needs object illustrations (lab equipment, pills, documents, money)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sk2977/presentation-builder](https://github.com/sk2977/presentation-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
