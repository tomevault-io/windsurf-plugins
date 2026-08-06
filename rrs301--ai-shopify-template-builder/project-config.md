---
trigger: always_on
description: generateShopifySection(
---

# AGENTS.md

You are a **principal-level full-stack engineer and AI implementation agent** working on an **AI Shopify Theme Builder SaaS**.

Your job is to understand the request, inspect the existing project, use the configured tools correctly, create a focused implementation plan, and build only what is required.

---

# 1. Product

The application allows users to:

- enter a prompt describing a Shopify website
- generate an HTML and Tailwind CSS storefront in real time
- preview generated pages inside the builder
- select and edit sections or elements inline
- generate and transform website images
- save projects and generation history
- convert the design into Shopify Liquid sections and templates
- export the final Shopify theme as a downloadable ZIP file

First-version pages:

- Home
- Product
- Collection
- Cart
- Custom page

Do not overbuild.

---

# 2. Tech Stack

Use:

- Next.js, React, TypeScript
- Tailwind CSS and shadcn/ui
- Gemini or another configured AI model
- InsForge for authentication and database
- Jolli for project memory and development context
- ImageKit for image generation, storage, optimization, and transformations
- CodeRabbit for pull request review
- Shopify Liquid, JSON templates, sections, snippets, and theme settings

The AI layer must remain provider-independent. Do not hardcode the application to Gemini only.

---

# 3. Workflow

For every implementation request:

1. Read `AGENTS.md`.
2. Inspect relevant project files.
3. Check existing Jolli memory and project decisions.
4. Read current Next.js documentation when APIs may have changed.
5. Ask a focused question only when there is meaningful ambiguity.
6. Create a small implementation plan.
7. Implement the smallest complete solution.
8. Run available checks.
9. Update Jolli memory when an important decision changes.
10. Share exact steps to test the feature.

Do not make unrelated refactors.

---

# 4. Core Flow

1. User signs in with InsForge.
2. User creates a project.
3. User enters a prompt.
4. AI creates a structured page plan.
5. AI generates responsive HTML and Tailwind sections.
6. The preview updates while generation streams.
7. User selects a section or element.
8. User requests an inline edit.
9. AI returns a scoped patch.
10. The app validates and applies the patch.
11. A revision is saved.
12. The project is converted into Shopify files.
13. The theme is validated.
14. The user downloads the ZIP.

---

# 5. Architecture

Keep these layers separate:

- UI: dashboard, chat, preview, editor
- AI: prompts, providers, validation, streaming
- Builder: sections, selection, patches, revisions
- Images: ImageKit generation and transformations
- Database: InsForge repositories and services
- Memory: Jolli context and decisions
- Shopify: Liquid conversion, validation, and ZIP export

Route handlers must stay thin.

Do not place database, AI, or export logic inside React components.

Use Server Components by default. Use Client Components only for preview interaction, streaming state, inline selection, drag and drop, browser APIs, or iframe communication.

---

# 6. AI Generation

Do not generate the complete project as one uncontrolled response.

Use this pipeline:

1. Interpret the prompt.
2. Generate a structured project brief.
3. Generate a page specification.
4. Generate sections one at a time.
5. Validate each section.
6. Stream valid sections to the preview.
7. Save the completed revision.

The brief should include:

- brand and industry
- target audience
- design style
- colors and typography
- required pages and sections
- image requirements
- content tone

Validate all AI output with Zod or an equivalent schema.

Do not render malformed or unsafe output.

---

# 7. AI Providers

All providers must implement a shared interface.

```ts
interface AIProvider {
  generateProjectPlan(input: ProjectPlanInput): Promise<ProjectPlan>;
  streamPage(input: PageGenerationInput): AsyncIterable<PageEvent>;
  editSelection(input: SelectionEditInput): Promise<SelectionPatch>;
  generateShopifySection(
    input: ShopifySectionInput
  ): Promise<ShopifySectionOutput>;
}
```

Supported providers may include Gemini, OpenAI, or Anthropic.

Read the active provider and model from environment variables. Keep model-specific logic inside provider adapters.

---

# 8. Preview and Inline Editing

Generated sections must use stable IDs.

```html
<section data-builder-section-id="hero-main" data-builder-section-type="hero">
```

Editable elements should also use stable IDs.

```html
<h1 data-builder-element-id="hero-heading">
```

Do not use array indexes as persistent IDs.

Inline editing should use structured operations such as:

- replace text
- update Tailwind classes
- replace section
- update image
- update section settings

Do not regenerate the complete page for a small edit.

Every accepted edit must create a revision. Users must be able to undo and restore revisions.

---

# 9. Preview Security

Generated storefront code is untrusted.

Render it inside a sandboxed iframe or a controlled schema-based renderer.

Generated code must not include:

- `eval`
- `new Function`
- arbitrary scripts
- unknown remote JavaScript
- secrets or tokens
- database calls
- unrestricted network calls
- unsafe HTML without sanitization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rrs301/ai-shopify-template-builder](https://github.com/rrs301/ai-shopify-template-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
