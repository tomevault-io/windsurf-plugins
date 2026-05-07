---
trigger: always_on
description: 37 specialized RevOps agents organized into 9 categories — 5 sales agents and 32 marketing agents. Each agent is an expert in its domain with deep frameworks, discovery questions, and structured output formats.
---

# Maestro Agent Roster

37 specialized RevOps agents organized into 9 categories — 5 sales agents and 32 marketing agents. Each agent is an expert in its domain with deep frameworks, discovery questions, and structured output formats.

## How Agents Work

- **Orchestrator routes automatically**: Describe what you need and the orchestrator sends you to the right specialist.
- **Direct access**: If you know which agent you need, talk to it directly.
- **Lateral delegation**: Any agent can consult any other agent via `delegate_to_agent`.
- **Reference docs**: Agents load deep reference materials on demand for comprehensive context.

## Agent YAML Structure

```yaml
name: agent-name
category: category-name
description: >
  What the agent does.
  Trigger: "phrase 1", "phrase 2", "phrase 3"

model:
  provider: anthropic
  name: claude-sonnet-4-20250514
  temperature: 0.4
  max_tokens: 8192

system_prompt: |
  Deep specialized prompt with frameworks, methodologies, output formats.

references:              # Loaded on demand into context
  - category/doc-name.md

tools:                   # Available tool integrations
  - calculator
  - delegate_to_agent

related_agents:          # Cross-referencing for lateral delegation
  - other-agent-name

max_tool_level: medium
```

---

## Sales (5 agents)

### pipeline-manager
Pipeline health, deal progression, revenue forecasting, and stalled deal identification.

**Triggers:** "pipeline review", "deal pipeline", "revenue forecast", "stalled deals", "pipeline health"

**Frameworks:** Pipeline Health Scorecard, Deal Velocity, Pipeline Coverage Ratio (3x rule), At-Risk Deal Identification

**Tools:** crm_deals, crm_pipeline, crm_activities, calculator

**Related agents:** deal-coach, lead-qualifier, prospector, analytics-interpreter

### lead-qualifier
Lead scoring, qualification assessment, and routing based on ICP fit and buying signals.

**Triggers:** "qualify this lead", "lead scoring", "is this a good lead", "ICP fit"

**Frameworks:** BANT Assessment (scored 1-5), ICP Fit Matrix (scored 0-2), Lead Source Quality, Qualification-to-Routing

**Tools:** crm_contacts, crm_companies, crm_deals, crm_activities

**Related agents:** prospector, pipeline-manager, deal-coach, email-sequence

### deal-coach
Deal strategy, objection handling, and next-step coaching for specific deals.

**Triggers:** "help with this deal", "deal strategy", "objection handling", "how to close", "deal stuck"

**Frameworks:** MEDDIC Deal Assessment, Objection Handling (LAER), Stakeholder Mapping, Stage-Specific Playbooks

**Tools:** crm_deals, crm_contacts, crm_companies, crm_activities

**Related agents:** pipeline-manager, meeting-prep, lead-qualifier

### meeting-prep
Pre-meeting research briefs compiling contact history, company context, deal status, and talking points.

**Triggers:** "meeting prep", "prepare for call", "pre-call research", "brief me on", "I have a meeting with"

**Frameworks:** Meeting Type Playbooks (discovery/demo/proposal/negotiation/check-in), Activity History Analysis

**Tools:** crm_contacts, crm_companies, crm_deals, crm_activities

**Related agents:** deal-coach, lead-qualifier, prospector

### prospector
Outbound prospecting strategy, target account identification, and outreach sequence design.

**Triggers:** "prospecting", "outbound strategy", "find leads", "target accounts", "cold outreach"

**Frameworks:** Target Account Selection, Multi-Touch Outreach Sequence (14-day), Personalization Tiers, First-Touch Email Angles

**Tools:** crm_companies, crm_contacts, crm_deals, crm_activities

**Related agents:** lead-qualifier, meeting-prep, email-sequence, linkedin-content

---

## Content & Copy (6 agents)

### blog-writer
Long-form posts, articles, and thought leadership content with SEO integration.

**Triggers:** "write a blog post", "article", "long-form content", "thought leadership piece"

**Frameworks:** Problem-Solution, Listicle, Ultimate Guide, Data-Driven, Narrative

**Related agents:** seo-auditor, keyword-researcher, newsletter-writer

### email-sequence
Automated email flows — welcome, nurture, sales, re-engagement, and onboarding sequences.

**Triggers:** "email sequence", "drip campaign", "welcome series", "nurture flow"

**Frameworks:** Welcome (7 emails), Nurture, Sales, Launch, Re-engagement, Onboarding

**Related agents:** sequence-builder, landing-page, segmentation

### landing-page
High-converting landing page copy with hero sections, features, social proof, and CTAs.

**Triggers:** "landing page", "sales page", "conversion copy", "page copy"

**Frameworks:** 7-Dimension CRO Analysis, Above-the-fold hierarchy, Section-by-section copywriting

**Related agents:** page-optimizer, signup-flow, social-content

### social-content
Platform-native social posts adapted for each platform's format and culture.

**Triggers:** "social media post", "social content", "Instagram caption", "social copy"

**Frameworks:** Platform-specific formats, Hook formulas, Engagement optimization

**Related agents:** linkedin-content, twitter-content, video-script

### video-script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaestroAgent/maestro](https://github.com/MaestroAgent/maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
