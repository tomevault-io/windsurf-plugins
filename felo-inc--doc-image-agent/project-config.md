---
trigger: always_on
description: Automatically illustrate Markdown documents by turning image markers into screenshots or generated images, then writing an image-enriched Markdown output. Use this skill when a document needs screenshots, AI-generated visuals, image placement, or end-to-end document illustration automation.
---


# Doc Image Agent

`doc-image-agent` is a single entry-point skill for automatically adding images to Markdown documents.

It supports:
- browser screenshots for product pages, dashboards, docs sites, and web apps
- AI-generated images for conceptual illustrations
- incremental reruns and partial regeneration
- semantic placement of images into the correct paragraph or section
- structured output directories for reusable assets and final Markdown

This package is intentionally published as **one main skill** plus supporting reference documents:
- `{baseDir}/references/browser-automation.md`
- `{baseDir}/references/playwright-mcp.md`
- `{baseDir}/references/site-explorer.md`
- `{baseDir}/references/image-generation.md`

Load this skill whenever the user asks to:
- add images to a Markdown article
- process a case file with image markers
- capture screenshots for documentation
- generate article visuals and insert them into a document
- rerun or fix image placement in an already processed document

## What This Skill Produces

Input:
- a Markdown document containing image markers and optionally an `Image Summary` table

Output:
- captured screenshots in a raw folder
- final image assets ready for Markdown references
- a generated README with image metadata
- an illustrated Markdown file with image markers replaced by real image references

## Project Root

All input, output, and cache paths are relative to a single project root directory (`{project-root}`).

At the very beginning of every run, **ask the user** which directory to use as the project root. If the user declines or says they have no preference, default to `/tmp/doc-image-agent`.

Once confirmed, **all subsequent paths** in this skill (cases/, output/, .cache/, etc.) resolve under `{project-root}/`.

## Recommended Directory Layout

```text
{project-root}/
├── cases/
│   └── {article-id}.md
├── output/
│   ├── {article-id}/
│   │   ├── raw/
│   │   │   ├── A1_example.png
│   │   │   └── A2_example.png
│   │   ├── A1_example.png
│   │   ├── A2_example.png
│   │   └── README.md
│   └── markdowns/
│       └── {article-id}.md
└── .cache/
    └── screenshots/
        └── {article-id}/
```

Conventions:
- `{project-root}/cases/` stores the source Markdown file.
- `{project-root}/output/{article-id}/raw/` stores original browser screenshots and should never be overwritten by later processing.
- `{project-root}/output/{article-id}/` stores final images referenced by Markdown.
- `{project-root}/output/markdowns/` stores the final illustrated Markdown.
- `{project-root}/.cache/screenshots/` stores reusable screenshot cache entries.

If the user specifies a different layout, follow the user instruction instead.

## Credentials

Some sites require authentication before the requested screenshot can be captured.

Read website credentials from environment variables using this pattern:

```text
PLAYWRIGHT_CRED_{SERVICE}_{FIELD}
```

Examples:
- `PLAYWRIGHT_CRED_FELO_EMAIL`
- `PLAYWRIGHT_CRED_FELO_PASSWORD`

Rules:
- read credentials from the environment instead of hardcoding them
- never print secrets back to the user
- if credentials are missing, tell the user which variable names are required
- if the workflow reaches a login, signup, registration, invite, verification, or onboarding gate that needs user-specific information, stop and ask the user how to proceed
- do not create new accounts, accept invitations, solve email verification, or invent profile information without explicit user input
- after the user provides credentials or instructions, continue from the interrupted step instead of restarting the whole run unless the user asks for a fresh run

## Supported Marker Formats

This skill must support both inline markers and summary tables.

### Format A: Heading-Based Screenshot Marker

```markdown
### 📷 Screenshot: {marker-id} ({filename})
Use: {why this screenshot exists}
Processing: {post-processing instruction}
Difference: {optional distinction from similar screenshots}
```

Fields:
- `marker-id`: unique screenshot identifier such as `A1`, `B3-1`, or `D3`
- `filename`: base filename without the marker prefix
- `Use`: what the screenshot should communicate
- `Processing`: crop, resize, or other post-processing needs
- `Difference`: optional explanation for how this screenshot differs from similar ones

### Format B: HTML Comment Image Marker

Screenshot:

```markdown
<!-- IMAGE: screenshot (https://example.com/app)
Description: Workspace dashboard showing project activity and team sidebar
Filename: workspace-dashboard.png
-->
```

Generated image:

```markdown
<!-- IMAGE: generated
Description: Editorial illustration of a collaborative AI workflow with folders and browser windows
Filename: ai-workflow-hero.png
-->
```

### Image Summary Table

A document may end with a summary table listing all required images:

```markdown
## Image Summary

| # | Type | Description | Filename |
|---|------|-------------|----------|
| 1 | generated | Description... | `hero.png` |
| 2 | screenshot | Description... | `dashboard.png` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Felo-Inc/doc-image-agent](https://github.com/Felo-Inc/doc-image-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
