---
trigger: always_on
description: Source for [developers.deepl.com](https://developers.deepl.com/), built on [Mintlify](https://mintlify.com/). Content lives in `.mdx` files under `docs/` (guides) and `api-reference/` (API reference).
---

# DeepL Developer Documentation

Source for [developers.deepl.com](https://developers.deepl.com/), built on [Mintlify](https://mintlify.com/). Content lives in `.mdx` files under `docs/` (guides) and `api-reference/` (API reference).

## Writing Style

### Voice and Tone

Write like a knowledgeable colleague, not a textbook. Clear, direct, and friendly. Use second person ("you") to address the reader. "We" is fine both for DeepL as a company ("We recommend...") and as inclusive "you and I" in tutorials ("Let's configure..."). Active voice, present tense ("returns" not "will return," per the Google Developer Documentation Style Guide). Be concise: every sentence earns its place. Cut filler and throat-clearing.

Avoid jargon unless the audience already knows it. Define technical terms on first use. Don't be overly formal ("hereafter", "aforementioned") or overly casual ("gonna", "super easy").

### Developer Documentation Tone

No marketing language or sales copy. Flag "best-in-class", "game-changing", "major step forward." Replace with technical descriptors. Developers reading these docs have already chosen DeepL or been directed here; describe what the API does and how to use it, don't pitch business outcomes.

Frame version comparisons neutrally. Focus on what the new version provides, not what the old version "lacked." Never say a previous version was "bad" or "broken."

Action-oriented descriptions: frontmatter `description` fields should tell developers what they'll learn. "Learn how to X" over "Information about X." Descriptions must be uniquely descriptive of the specific page content — never generic enough to apply to multiple pages.

When noting limitations or incompatibilities, mention if they're temporary. "Currently only compatible with X. Support for Y will be added in a future update." Avoid indefinite phrases like "for the time being" without specifics.

Write from the external developer's perspective, never from DeepL's internal perspective. The reader doesn't care about internal team names, decisions, or how a feature was built. Watch for: "We will..." / "We plan to..." (rewrite as what the developer can expect), internal team references, Jira tickets, explaining DeepL's decision-making instead of the outcome.

### Structure and Formatting

Front-load key information. Lead with what the reader needs to do, not background.

Short paragraphs (3-4 lines max). Use headings, bullets, and tables to break up walls of text, but don't add subsections just for organization if 2-3 paragraphs or bullets suffice.

Use tables for structured comparisons (feature matrices, parameter lists, method trade-offs).

Use Mintlify components (`<Tabs>`, `<Steps>`, `<Tip>`, `<Warning>`, `<Note>`) where they improve clarity. Don't overuse them. Consolidate related examples into tabs rather than separate sections, placing the most common use case first.

### Content Principles

Write for an external developer who has never used DeepL before, wants to integrate a translation/writing API into their product, will skim your page in 30 seconds, and needs to know what to do, not how DeepL works internally.

Task-oriented over feature-oriented. Frame docs around what the user wants to accomplish. Every piece of information should answer "so what should I do?" If it doesn't, add guidance.

Show what a feature is before announcing what changed. Never assume the reader knows what something is. When describing updates or configuration, include one sentence explaining what the feature does and why a developer would use it.

Warn about the mistakes developers will actually make. Think about what goes wrong in practice. Add warnings for common pitfalls, not theoretical ones.

Progressive disclosure: introduce simple concepts before complex ones. Concrete before abstract: show examples before deep technical details. Connect technical features to practical use cases.

Show, don't just tell. Pair explanations with concrete examples.

Link generously. Cross-reference related pages rather than duplicating content.

When presenting multiple approaches, include a comparison table and "when to use this" guidance for each.

In migration guides and technical sections, prefer concise technical bullets over explanatory prose.

### Lists and Bullets

Avoid excessive use of bulleted lists. When you do use them:
- Bullet points end without periods (unless multi-sentence). Capitalize standalone items; lowercase is fine when bullets complete a stem sentence
- All items in a list follow the same grammatical structure (parallel construction)
- Use `-` or `*` for unordered lists (either is fine, but be consistent within a page), numbers for sequential steps

### Cross-references and Links

Use relative links for internal docs. Link text should be descriptive: "See [Managing API Keys](/docs/getting-started/managing-api-keys)" not "See [here](/docs/getting-started/managing-api-keys)."

### Formatting

No em-dashes. Use commas, periods, or parentheses. No curly/smart quotes; use straight quotes. Consistent list markers within a page (`-` or `*`, either is fine).

## Code Examples

### Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeepL/api-docs](https://github.com/DeepL/api-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
