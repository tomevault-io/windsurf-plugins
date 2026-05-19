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

## 🔍 CRITICAL: Always Verify Implementation Before Documenting

**NEVER assume how Igniter.js APIs work. ALWAYS verify the actual implementation.**

Before documenting any feature, API, or pattern:

1. **Search the codebase** for the actual implementation in `packages/`
2. **Read the source code** to understand:
   - Function signatures and parameters
   - Return types and structures
   - Available methods and properties
   - Type definitions in `.d.ts` files
3. **Check interfaces** (`interface` and `type` definitions) to understand contracts
4. **Review existing examples** in the codebase for correct usage patterns
5. **Test your understanding** by looking at how it's used elsewhere

### Why This Matters

- **Type Safety**: Incorrect API usage breaks TypeScript inference
- **Developer Trust**: Wrong documentation causes frustration and bugs
- **Code Consistency**: Ensures all examples match the actual implementation
- **Framework Evolution**: APIs change, and documentation must reflect current state

### Verification Checklist

- [ ] Found the implementation in `packages/`
- [ ] Read the source code and interfaces
- [ ] Verified all imports and exports
- [ ] Checked type definitions
- [ ] Reviewed how it's used in existing code/docs
- [ ] Ensured code examples compile and work correctly

## For LLM Content Generation

### Formatting Instructions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipebarcelospro/igniter-js](https://github.com/felipebarcelospro/igniter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
