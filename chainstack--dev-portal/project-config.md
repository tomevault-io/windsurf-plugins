---
trigger: always_on
description: This repository contains the Chainstack Developer Portal built with Mintlify. When working on the project interactively with an agent (e.g. the Codex CLI) please follow the guidelines below for efficient documentation development and maintenance.
---

# AGENTS Guidelines for This Repository

This repository contains the Chainstack Developer Portal built with Mintlify. When working on the project interactively with an agent (e.g. the Codex CLI) please follow the guidelines below for efficient documentation development and maintenance.

## 1. Use Mintlify Development Server

* **Always use `mintlify dev`** for local preview.
* **Test at** http://localhost:3000.
* **Do _not_ deploy to production** during agent development sessions.
* **Run link checks** with `mint broken-links` before submitting PRs.

## 2. Two-product architecture

The docs use Mintlify's product switcher with two products:

- **Cloud** — managed blockchain infrastructure (docs in `docs/`)
- **Self-Hosted** — deploy on your own infrastructure (docs in `docs/self-hosted/`)

Both products share the same `docs.json` for navigation. When adding pages, make sure you add them under the correct product.

## 3. Keep documentation structure consistent

When creating new documentation:

1. Install Mintlify CLI: `npm i -g mintlify`
2. Run dev server: `mintlify dev`
3. Check for broken links: `mint broken-links`
4. Validate build: `mint validate`
5. Follow existing patterns for consistency

## 4. MDX file requirements

Every MDX file must start with YAML frontmatter:

```yaml
---
title: "Clear, specific title"
description: "Concise description for SEO and navigation"
---
```

Never forget to add new files to `docs.json` navigation.

## 5. Navigation management

When adding new documentation:

1. **Create MDX file** in appropriate directory (`docs/`, `reference/`, or `recipes/`)
2. **Add frontmatter** with title and description
3. **Update `docs.json`** to include file in navigation (without `.mdx` extension)
4. **Test navigation** works in local preview

## 6. Release notes workflow

The documentation has two products with separate release notes:

- **Cloud** — managed blockchain infrastructure (`changelog.mdx` + `changelog/` directory)
- **Self-Hosted** — deploy on your own infrastructure (`docs/self-hosted/release-notes.mdx` + `docs/self-hosted/changelog/`)

### Cloud release notes

Creating release notes requires three steps:

#### Step 1: Update `changelog.mdx`
Copy previous entry within `<Update>` tags and paste on top:
```mdx
<Update label="Chainstack updates: December 31, 2025" description=" by Your Name" >

**Protocols**. Your update content here.

<Button href="/changelog/chainstack-updates-december-31-2025">Read more</Button>
</Update>
```

#### Step 2: Create changelog file
Create `changelog/chainstack-updates-december-31-2025.mdx` with same content (without `<Update>` tags).

#### Step 3: Update navigation
In `docs.json`, in the Cloud product's `Release notes` tab, add the newly created file name (without `.mdx`) between `changelog` and the previous release notes entries:
```json
"pages": [
  "changelog",
  "changelog/chainstack-updates-december-31-2025",
  "changelog/chainstack-updates-previous-entry"
]
```

### Self-Hosted release notes

#### Step 1: Update `docs/self-hosted/release-notes.mdx`
Add a new `<Update>` entry at the top:
```mdx
<Update label="Chainstack Self-Hosted v1.0.0: January 28, 2026" description="">

**Initial beta release**. Your update content here.

<Button href="/docs/self-hosted/changelog/chainstack-self-hosted-v1-0-0-january-28-2026">Read more</Button>
</Update>
```

#### Step 2: Create changelog file
Create file in `docs/self-hosted/changelog/` using format `chainstack-self-hosted-v1-0-0-month-day-year.mdx`. The page title should match the label.

#### Step 3: Update navigation
In `docs.json`, in the Self-Hosted product's `Release notes` tab, add the new changelog page.

## 7. Mintlify components usage

Use appropriate components for content:

| Component | Use Case |
| --------- | -------- |
| `<Note>` | Helpful supplementary information |
| `<Warning>` | Important cautions and breaking changes |
| `<Tip>` | Best practices and expert advice |
| `<Info>` | Neutral contextual information |
| `<Check>` | Success confirmations |
| `<Steps>` | Sequential instructions |
| `<CodeGroup>` | Multiple language examples |
| `<Tabs>` | Side-by-side information |
| `<Accordion>` | Progressive disclosure |
| `<Card>`, `<CardGroup>` | Highlighting content |
| `<Frame>` | Wrapping images with alt text |

## 8. Code examples

When adding code examples:

* Include complete, runnable examples
* Use `<CodeGroup>` for multiple languages
* Specify language tags on all code blocks
* Include realistic data, not placeholders
* Use `<RequestExample>` and `<ResponseExample>` for API docs

## 9. Writing style guidelines

Follow the established style guide:

* **Use sentence case** for everything (not Title Case)
* **Active voice** over passive voice
* **American English** spelling
* **Oxford comma** in lists
* **Escape dollar signs** with backslash (`\$`) in MDX files
* **Bold UI elements** when referencing them
* Use `>` for UI clickthrough paths

## 10. API documentation

For API reference documentation:

* Document all parameters with `<ParamField>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chainstack/dev-portal](https://github.com/chainstack/dev-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
