---
trigger: always_on
description: This is a technical book for engineers, data people, and technical operators building technology at VC funds. It's opinionated, practical, and based on real experience building VC infrastructure at Inflection and EQT.
---

# Building for Venture Capital - Content Guidelines

## Project Overview

This is a technical book for engineers, data people, and technical operators building technology at VC funds. It's opinionated, practical, and based on real experience building VC infrastructure at Inflection and EQT.

## Writing Voice and Tone

**Direct and opinionated**: This book takes strong positions based on real experience. Use clear language:

- ✅ "Don't build your own CRM. Use Attio or Affinity."
- ❌ "You might want to consider whether building a CRM is the right choice for your organization."

**Second-person "you"**: Always address the reader directly as someone doing this work.

- ✅ "You're building for 5-30 people..."
- ❌ "One might build for teams of 5-30 people..."
- ❌ "Engineers build for teams of 5-30 people..."

**Technical but accessible**: Assume technical competence but explain context. The reader is a competent engineer who may not know VC-specific details.

**No corporate speak or buzzwords**: Avoid marketing language, hype, and vague business terminology.

- ✅ "Use Postgres. It's boring technology that works."
- ❌ "Leverage cutting-edge database solutions to maximize operational synergies."

**Anti-hype**: Call out trends that don't matter. Be skeptical of new technology unless there's a clear practical benefit.

## Content Principles

**Practical over theoretical**: Every section should answer "what should I actually do?" Include real examples, specific tools, actual code.

**Acknowledge trade-offs**: Most decisions have context. Explain when advice applies and when it doesn't.

- "For funds under 50 people, use [X]. Larger funds might need [Y]."
- "This works if you're technical. If you're not, buy [vendor] instead."

**Real examples from experience**: Reference actual work at Inflection and EQT when relevant. Be specific:

- ✅ "At Inflection, we use Attio for CRM and sync it to Postgres nightly."
- ❌ "Some funds use CRM systems and might integrate them with databases."

**Strong opinions, weakly held**: Take clear positions but acknowledge alternatives and admit uncertainty when it exists.

## Structure and Formatting

**Start sections with clear thesis statements**: First sentence should tell the reader what they'll learn or what position you're taking.

**Use "The bottom line" summaries**: Many chapters end with a "Bottom Line" section that distills key takeaways. These should be practical, action-oriented summaries.

**Author Notes in `<Tip>` components**: Use for personal asides, contextual notes, or clarifications:

```mdx
<Tip>
  **Author note**: At Inflection, we tried building this ourselves and it was a mistake. Save
  yourself the pain.
</Tip>
```

**Code examples with context**: Always include language tags. Explain what the code does and why, not just what it is:

```typescript
// TypeScript with Zod for validation
import { z } from "zod"

const CompanySchema = z.object({
  name: z.string(),
  // ... rest of schema
})
```

**Real examples clearly labeled**: When providing case studies or specific examples:

```mdx
**Example: At Inflection, we use Mistral for PDF reading**

When processing pitch decks...
```

## Style Specifics

**Sentence structure**: Prefer shorter sentences and clear paragraphs. Break up long blocks of text. Use periods over em-dashes in most cases.

**Lists for clarity**: Use bulleted or numbered lists to make information scannable:

- When listing tools or options
- When outlining steps in a process
- When comparing approaches

**Avoid passive voice**: Write actively.

- ✅ "Use Postgres for your data warehouse"
- ❌ "Postgres should be used for data warehousing"

**Be precise with technical terms**: Use correct terminology. Don't say "database" when you mean "data warehouse". Don't say "AI" when you mean "LLM".

**Code over prose**: When showing how to do something, include actual code examples rather than just describing in prose.

## What This Book Is Not

**Not comprehensive**: Cover what matters for VC funds. Skip edge cases that don't apply to the audience.

**Not vendor-neutral**: Recommend specific tools that work. Don't try to cover every option.

**Not future-focused**: Focus on what works today. Acknowledge emerging trends when relevant, but don't speculate about technology that doesn't exist yet.

**Not academic**: This is practitioner knowledge. Cite specific experiences over research papers.

## Content Strategy

**Evergreen when possible**: Focus on principles and approaches that will remain relevant as specific tools change.

**Update for accuracy**: Keep tool recommendations current. If vendor landscape changes, update recommendations.

**Real problems first**: Start from problems VC funds actually face, not from technology looking for applications.

## Technical Standards

**Format**: MDX files with YAML frontmatter

- title: Clear, descriptive (e.g., "Data Modeling and Schema Design")
- description: Concise, practical summary (e.g., "How to model companies, deals, and relationships - the core data structures for VC infrastructure")


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexpatow/building-for-vc](https://github.com/alexpatow/building-for-vc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
