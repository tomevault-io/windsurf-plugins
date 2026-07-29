---
trigger: always_on
description: This guide ensures consistent, high-quality documentation across all Igniter.js content types (blog, docs, templates, updates). It's designed for both human authors and AI agents to produce excellent developer experiences.
---

# Writing Guidelines for Humans and LLMs

This guide ensures consistent, high-quality documentation across all Igniter.js content types (blog, docs, templates, updates). It's designed for both human authors and AI agents to produce excellent developer experiences.

## Core Style Principles

| Principle | Definition | Example: Good | Example: Bad |
|-----------|------------|--------------|-------------|
| Clear Language | Use simple, direct language with short words and sentences. Avoid unnecessary jargon. | "Click save to store your changes." | "Initiate the persistence process by activating the storage mechanism." |
| Developer-Focused | Write in a professional but approachable tone that addresses the reader directly. | "You can configure this option in the settings panel." | "Users might want to adjust configuration parameters." |
| Example-Driven | Include practical code examples for all key concepts. | "Use the `useState` hook: `const [count, setCount] useState(0)`" | "State management is an important React concept." |
| Active Voice | Make the subject perform the action rather than receive it. | "React renders the component." | "The component is rendered by React." |
| Success-First | Show working examples before explaining theory. | "First, create a component: `function Button() {...}`. Now let's understand how it works..." | "The component lifecycle has several phases which you must understand before implementation..." |
| Transparent | Be honest about limitations and challenges. | "This approach works well for small datasets but may cause performance issues with larger ones." | "This is the optimal solution for data management." |
| Consistent Terms | Use the same terminology throughout to refer to the same concepts. | "Route" consistently or "endpoint" consistently, not both interchangeably. | Mixing "callback function" and "handler" for the same concept. |

---

## Fumadocs-Specific Guidelines

Igniter.js documentation uses **Fumadocs**, a modern documentation framework with MDX support and powerful components.

### Available MDX Components

Always use these components from `mdx-components.tsx` when appropriate:

#### Content Organization
- **`<Callout />`** - Highlight important information (types: `info`, `warn`, `error`, `success`)
- **`<Accordions />` + `<Accordion />`** - Collapsible content sections
- **`<Tabs />` + `<Tab />`** - Tabbed content (supports persistence with `groupId`)
- **`<Steps />` + `<Step />`** - Step-by-step instructions

#### Code & Files
- **`<CodeBlock />`** - Enhanced code blocks with syntax highlighting
- **`<Files />`, `<Folder />`, `<File />`** - File structure visualization
- **`<TypeTable />`** - API/type documentation tables

#### Other Components
- **`<Banner />`** - Page-level announcements
- **`<Card />`, `<Cards />`** - Content cards for links/features

### Component Usage Examples

```mdx
<!-- Callout for warnings -->
<Callout type="warn" title="Important">
  Make sure to configure your environment variables before deployment.
</Callout>

<!-- Steps for tutorials -->
<Steps>
  <Step>
    ### Install Dependencies
    Run `npm install @igniter-js/core`
  </Step>
  
  <Step>
    ### Configure
    Create a `igniter.config.ts` file
  </Step>
</Steps>

<!-- Tabs for multiple options -->
<Tabs items={['npm', 'pnpm', 'yarn', 'bun']} groupId="package-manager">
  <Tab value="npm">
    ```bash
    npm install @igniter-js/core
    ```
  </Tab>
  <Tab value="pnpm">
    ```bash
    pnpm add @igniter-js/core
    ```
  </Tab>
</Tabs>

<!-- File structure -->
<Files>
  <Folder name="src" defaultOpen>
    <Folder name="features">
      <File name="users.controller.ts" />
    </Folder>
    <File name="igniter.ts" />
  </Folder>
</Files>
```

---

## For LLM Content Generation

### Formatting Instructions

When generating content with an LLM, ensure the following:

1. **Structured Data**: Present information in well-defined structures like tables, numbered lists, and hierarchical headings
2. **Explicit Examples**: Always include contrastive examples (good vs. bad)
3. **Clear Boundaries**: Use explicit section markers and consistent formatting patterns
4. **Context-Awareness**: Begin with the most critical information for the document type
5. **Pattern Consistency**: Maintain consistent patterns throughout similar sections
6. **Component Usage**: Use Fumadocs components appropriately for the content type

### LLM Content Templates

For each document type, LLMs should structure content as follows:

```yaml
DocumentType: [Tutorial|HowTo|Reference|Explanation|BlogPost|Update]
Audience: [Beginner|Intermediate|Advanced]
PrimaryGoal: "Single sentence describing document purpose"
Sections:
  - Name: "Introduction"
    Content: "Clear goal statement with outcomes"
  - Name: "Prerequisites"
    Content: "Bulleted list of requirements"
  - Name: "MainContent"
    Content: "Follows appropriate structure for document type"
  - Name: "Conclusion"
    Content: "Summary and next steps"
```


## Document Types and Their Styles

### Documentation (Docs)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipebarcelospro/igniter-js](https://github.com/felipebarcelospro/igniter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
