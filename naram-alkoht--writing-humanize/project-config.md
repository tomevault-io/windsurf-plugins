---
trigger: always_on
description: Remove signs of AI-generated writing from text while preserving the author's voice, meaning, and domain-specific vocabulary. Use this skill whenever the user asks to "humanize" writing, make text "sound more natural," "remove AI patterns," or "make it sound like me." Works across any domain — technical writing, marketing copy, blog posts, case studies, academic writing, documentation, and more.
---


# Writing Humanize

Transform AI-generated or AI-assisted text into natural, human-sounding writing — without rewriting content, simplifying language, or changing meaning.

## Core Philosophy

Humanizing is about removing **structural and stylistic AI fingerprints**, not about rewriting content. The goal is text that reads like a knowledgeable person wrote it from experience, not like a model generated it from a prompt.

Two principles govern every change:

1. **Minimum effective intervention.** Fix only what triggers an AI-detection reflex. Don't touch what already sounds natural.
2. **Preserve the author's vocabulary.** Never replace domain-specific terms with casual synonyms. A legal writer's "indemnification" stays "indemnification." A developer's "isolate tenant data" stays "isolate tenant data." A doctor's "contraindicated" stays "contraindicated." Precise vocabulary exists because it's precise — changing it makes the writing less accurate, not more human.

---

## The 25 Patterns to Fix

Scan the text for these patterns. Fix only what's present.

---

### Content Patterns

**1. Significance Inflation**
AI elevates ordinary facts into historic milestones.

- Before: "marking a pivotal moment in the evolution of..."
- After: state the actual fact plainly — "was established in 1989 to collect regional statistics"

**2. Notability Name-Dropping**
AI lists prestigious outlets without context, as if mention alone confers credibility.

- Before: "cited in NYT, BBC, FT, and The Hindu"
- After: anchor the reference — "In a 2024 NYT interview, she argued..."

**3. Superficial -ing Analyses**
AI uses present participles to imply meaning without stating it.

- Before: "symbolizing... reflecting... showcasing..."
- After: remove entirely, or replace with a specific claim backed by a source

**4. Promotional Language**
AI describes things the way a brochure would.

- Before: "nestled within the breathtaking region"
- After: "is a town in the Gonder region"

**5. Vague Attributions**
AI cites "experts" or "studies" without specifics.

- Before: "Experts believe it plays a crucial role"
- After: "according to a 2019 survey by..." — or remove the claim entirely

**6. Formulaic Challenges**
AI acknowledges difficulty with a generic pivot to optimism.

- Before: "Despite challenges... continues to thrive"
- After: name the actual challenge with specific facts

---

### Language Patterns

**7. AI Vocabulary**
Certain words appear disproportionately in AI output.

Words to reduce: *additionally, testament, landscape, showcasing, crucial, vital, groundbreaking, transformative, pivotal, innovative, seamless, robust, comprehensive, leverage, delve, underscore, foster, navigate, realm, embark*

- Before: "Additionally, this serves as a testament to the transformative potential..."
- After: "It also shows that..."

**8. Copula Avoidance**
AI reaches for elaborate verb phrases when "is" and "has" would do.

- Before: "serves as... features... boasts... stands as..."
- After: "is... has"

**9. Negative Parallelisms**
AI contrasts two things using a "not just X, it's Y" construction.

- Before: "It's not just about autocomplete; it's about unlocking creativity at scale"
- After: state the actual point directly

**10. Rule of Three**
AI defaults to grouping things in threes, even when two or four would be more accurate.

- Before: "innovation, inspiration, and insights"
- After: use the natural number of items the content actually requires

**11. Synonym Cycling**
AI rotates synonyms to avoid repetition, creating artificial variety.

- Before: "protagonist... main character... central figure... hero" (all referring to the same person)
- After: pick the most accurate term and repeat it

**12. False Ranges**
AI uses "from X to Y" constructions that span everything, meaning nothing.

- Before: "from the Big Bang to dark matter"
- After: list the topics directly

---

### Style Patterns

**13. Em-Dash Overuse**
AI uses em-dashes heavily — often 3–5 per paragraph — creating a recognizable rhythm.

- Before: "The system — built with Laravel — handles multi-tenancy — using RLS — which ensures isolation."
- After: one or two em-dashes per paragraph maximum. Replace the rest with periods, commas, or parentheses.

**14. Boldface Overuse**
AI bolds terms indiscriminately, turning prose into a visual checklist.

- Before: "**OKRs**, **KPIs**, and **BMC** are central to the framework"
- After: "OKRs, KPIs, and BMC are central to the framework"

**15. Inline-Header Lists**
AI structures lists as bolded labels followed by descriptions, even when prose would flow better.

- Before: "**Performance:** Performance improved significantly. **Reliability:** Uptime increased."
- After: convert to prose — "Performance improved significantly, and uptime increased."

**16. Title Case Headings**
AI capitalizes every word in headers, even common words.

- Before: "Strategic Negotiations And Partnerships"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Naram-Alkoht/writing-humanize](https://github.com/Naram-Alkoht/writing-humanize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
