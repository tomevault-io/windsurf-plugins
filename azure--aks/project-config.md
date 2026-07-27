---
trigger: always_on
description: > **Scope**: File-specific patterns for blog post Markdown files.
---


# AKS Blog Post Content Guidelines

> **Scope**: File-specific patterns for blog post Markdown files.  
> **Module Context**: See `website/AGENTS.md` for architecture and workflows.  
> **Repo Standards**: See `.github/copilot-instructions.md` for general conventions.

## Front Matter (Required)

Every blog post MUST include:

```yaml
---
title: "Descriptive Post Title"
date: YYYY-MM-DD
description: "SEO-optimized summary (150-160 characters)"
authors: [author-key]  # From blog/authors.yml
tags: [tag1, tag2]     # From blog/tags.yml
draft: false            # Optional: set to true to hide from production
---
```

**Validation Checklist**:

- [ ] `title`: Clear, descriptive, under 60 chars
- [ ] `date`: Valid format, can be future-dated for future publishing
- [ ] `description`: 150-160 chars, includes keywords
- [ ] `authors`: Valid keys from `blog/authors.yml`
- [ ] `tags`: Valid keys from `blog/tags.yml`

### Blog Post Date

The publish date is determined by the **folder name** (e.g., `2025-12-19-my-post/index.md`). Docusaurus extracts the date from folder/filename patterns like `YYYY-MM-DD-slug` or `YYYY/MM/DD/slug`. If you specify a `date` in front matter, it **overrides** the folder-derived date.

**Important**: Unlike Jekyll, Docusaurus does **NOT** hide future-dated posts. All posts are published immediately when deployed, regardless of date. To prevent premature publishing:

- Use `draft: true` in front matter to exclude from production builds (visible only in dev)
- Use `unlisted: true` to hide from listings but keep accessible via direct link
- Alternatively, keep the post in a separate branch until ready

**Before merging**: If PR review takes longer than expected and the folder date becomes stale, update the folder name (or add a `date` override in front matter) to reflect the actual publish date. Posts should show the date readers will see them, not when they were originally drafted.

## Content Structure

### Required Pattern

```markdown
---
title: "..."
date: YYYY-MM-DD
description: "..."
authors: [...]
tags: [...]
---

[Opening paragraph: Hook the reader, set context, preview content]

<!-- truncate -->

![Hero Image](./hero-image.png)

## Section 1: Problem/Context
[Describe the challenge or background...]

## Section 2: Solution/Details
[Explain the approach or feature...]

## Section 3: Implementation
[Step-by-step guide or examples...]

## Conclusion
[Summarize key takeaways...]
```

### Critical Elements

1. **Truncation marker**: `<!-- truncate -->` after 2-3 intro paragraphs (shows on listing page)
2. **Hero image**: Use `./hero-image.png` for same-directory assets
3. **Heading hierarchy**: H2 (`##`) for major sections, H3 (`###`) for subsections
4. **Alt text**: All images MUST have descriptive alt text

## Writing Style

### Target Audience

- **Primary**: AKS users, Kubernetes practitioners, platform engineers
- **Level**: Intermediate to advanced technical knowledge
- **Expectation**: Practical, actionable content with examples

### Tone Guidelines

- ✅ Follows the Microsoft Style Guide
- ✅ Professional yet approachable
- ✅ Direct and concise
- ✅ Technical but accessible
- ❌ No marketing fluff or buzzwords
- ❌ No unexplained jargon
- ❌ No passive voice (prefer active)

### Content Requirements

- **Length**: 800-1500 words (longer acceptable if high-value)
- **Paragraphs**: 3-4 sentences max per paragraph
- **Lists**: Use bullets/numbers liberally to break up text
- **Examples**: Include code snippets, commands, or diagrams
- **Links**: External references to docs, GitHub, or related posts
- **Originality**: Must be original content (cite sources if adapted)

## Formatting Patterns

### Headings

```markdown
## Major Section (H2)

### Subsection (H3)

#### Detail Point (H4 - use sparingly)
```

**Rules**:

- Title (H1) auto-generated from front matter
- Use H2 for main sections
- Use H3 for subsections
- Avoid H4+ (indicates over-complexity)

### Code Blocks

**Always specify language**:

````markdown
```bash
kubectl get pods -n production
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
```

```typescript
export function handleRequest(): void {
  // Implementation
}
```
````

**Best Practices**:

- Include comments for complex code
- Show realistic examples (not `foo`/`bar`)
- Keep snippets under 30 lines (link to gists for longer)

### Links

```markdown
✅ [Azure Kubernetes Service documentation](https://learn.microsoft.com/azure/aks)
✅ See the [AKS roadmap](https://github.com/orgs/Azure/projects/685) for planned features.

❌ Click [here](https://example.com) for more info.
❌ Check out https://example.com
```

**Rules**:

- Descriptive link text (not "click here" or "this link")
- No bare URLs (always use `[text](url)` syntax)
- Prefer Microsoft Learn for Azure docs

### Images

```markdown
![Descriptive alt text explaining the image](./image-filename.png)
```

**Requirements**:

- Images in same directory as `index.md`
- Use relative paths: `./image.png`
- Descriptive alt text (accessibility + SEO)
- Optimize before commit (compress PNGs/JPGs)
- Max 500KB per image

### Emphasis

```markdown
**Important term or concept** (bold - use sparingly)
*Subtle emphasis* (italic - rare)
`code or technical term` (inline code)
```

### Blockquotes

```markdown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/AKS](https://github.com/Azure/AKS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
