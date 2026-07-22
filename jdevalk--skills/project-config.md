---
trigger: always_on
description: Detailed rewrite guidance for Phase 2 (Rewrite). Read this file when producing the rewritten readme.txt. The parent `SKILL.md` has the workflow and audit checklist.
---

# WordPress Plugin readme.txt Optimizer — Rewrite recipes

Detailed rewrite guidance for Phase 2 (Rewrite). Read this file when producing the rewritten readme.txt. The parent `SKILL.md` has the workflow and audit checklist.

## Rewrite rules

These rules apply to every rewrite:

- **Preserve all factual content** — don't invent features, screenshots, or version numbers that weren't in the original.
- **Improve, don't fabricate** — if a section is missing (e.g. no FAQ), write a placeholder with `<!-- TODO: add real questions from your support forum -->` rather than making things up.
- **Keep the exact readme.txt format** — WordPress.org uses a specific Markdown-like syntax. Follow it precisely (see Format Reference below).
- **Apply every audit finding** — the rewrite should address every issue flagged in Phase 1.
- **Keyword discipline** — use the primary keyword in: plugin name (if it fits), short description, first paragraph of long description, at least one FAQ question, and screenshot captions. Don't keyword-stuff; natural usage only.
- **Tone** — professional but human. Write for the person installing the plugin, not for the developer who built it.

---

## Section-by-section writing guidance

### Plugin Name & Tags

The plugin name line (`=== Plugin Name ===`) is the H1 of the listing page and carries the most search weight.

- If the original name is a brand-only name (e.g. "Flavius"), append a dash and a keyword descriptor: `=== Flavius – Contact Form Builder ===`. Keep the full string under ~60 characters.
- If the name already contains the primary keyword, leave it. Don't double up.
- Use exactly 5 tags. Pick the highest-volume terms that directly describe what the plugin does. Avoid meta-tags like "widget" or "admin" unless the plugin is literally a widget or admin tool.
- Order tags by relevance — the first tag carries slightly more weight.

### Short Description

This is the single highest-leverage string in the entire readme. It appears in search results and on the plugin card.

- **Lead with a benefit or outcome**, not the plugin name or a feature list. Bad: "A powerful plugin for forms." Good: "Add spam-free contact forms to any page in under a minute."
- Stay under 150 characters — hard limit, WordPress.org truncates after that. Aim for 120-140 to leave margin.
- Include the primary keyword naturally, ideally in the first half of the sentence.
- No period at the end (convention, not a rule — but most top plugins skip it).
- Avoid: "powerful", "easy", "best", "ultimate", "simple", "lightweight". These are noise words that every plugin uses.
- One concrete claim is worth more than three vague adjectives.

### Long Description

The long description is the body of the listing page. Structure it for scanning — most visitors won't read every word.

**First paragraph (the hook):**

- Lead with the user's problem or desired outcome, not the plugin's history or technical architecture.
- Include the primary keyword in the first 150 words.
- This paragraph should answer: "Why should I install this instead of the other five plugins that showed up in search?"

**Features section:**

- Use an H2 heading: `== Features ==` or `== What you get ==`.
- Present features as benefits with bullet points. Bad: `* SMTP support`. Good: `* **Reliable email delivery** — route messages through any SMTP server so nothing lands in spam.`
- Bold the benefit at the start of each bullet, then explain in plain text.
- Group related features under H3 subheadings if the list is long (8+ items).

**Social proof:**

- If the plugin has notable stats (active installs, star rating, notable users), weave them in. A standalone line like "Trusted by 50,000+ sites" near the top is more effective than burying it in a paragraph.
- Don't fabricate stats. If the original readme doesn't mention installs or reviews, leave this out.

**Call to action:**

- If there's a Pro version, docs site, or demo, link to it with a clear CTA. Use Markdown links: `[View the demo](https://example.com)`.
- Place the CTA after the features section, not before.

**Secondary keywords:**

- Weave them into headings, feature bullets, and the closing paragraph. Don't force them — if a keyword doesn't fit naturally, skip it.

**Formatting:**

- Use `== Heading ==` for H2, `= Subheading =` for H3 (WordPress.org readme syntax, not standard Markdown).
- Use `**bold**` for emphasis, backticks for code references.
- Keep paragraphs short — 2-4 sentences max. The listing page renders with narrow column width.

### Installation

Keep installation steps short and standard. Most plugins follow the same pattern.

**Automatic installation (always list first — it's what most users do):**

1. Go to Plugins > Add New in your WordPress dashboard.
2. Search for "[Plugin Name]".
3. Click Install Now, then Activate.

**Manual installation:**

1. Download the plugin ZIP from WordPress.org.
2. Go to Plugins > Add New > Upload Plugin.
3. Upload the ZIP file and click Install Now.
4. Activate the plugin.

**Post-activation steps:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdevalk/skills](https://github.com/jdevalk/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
