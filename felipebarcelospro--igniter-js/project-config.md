---
trigger: always_on
description: This guide defines the writing standards for all Igniter.js content: documentation, blog posts, templates, and updates.
---

## ✍️ Unified Documentation Style Guide (for LLMs & Authors)

This guide defines the writing standards for all Igniter.js content: documentation, blog posts, templates, and updates.

### 🎯 Core Style Principles (Applies to All Documentation)

1. **Clarity Over Cleverness**  
   Use simple, direct language. Favor short words and short sentences. Avoid jargon unless it's essential—and if it is, define it once.

2. **Speak Developer**  
   Write the way you'd speak to a smart, curious engineer sitting next to you. Friendly, confident, and precise.

3. **Show, Don't Just Tell**  
   Use examples liberally. Explain ideas with real code, not abstract theory. Every key concept should be backed by a working snippet.

4. **Write in Active Voice**  
   ✅ "Click the button to save your changes."  
   ❌ "The button should be clicked in order for changes to be saved."

5. **Present First, Explain Later**  
   Get the reader to success as quickly as possible. After they've seen something work, then explain how/why it works.

6. **Be Honest and Human**  
   If something is tricky, say so. If you're recommending a workaround, explain why. Transparency builds trust.

7. **Use consistent terminology**  
   Always refer to concepts, components, and APIs by the same name. Don't mix "endpoint" and "route" if they mean the same thing.

8. **Verify Before Documenting**  
   **NEVER assume API behavior.** Always check the actual implementation in `packages/` before writing code examples or documentation. Read source code, interfaces, and type definitions to ensure accuracy.

---

### 📘 Documentation (Docs): Writing Style

**Tone:** Clear, technical, and helpful  
**Voice:** An expert engineer sharing knowledge directly

#### ✅ Style Rules:
- **Verify Implementation**: Always check `packages/` source code before documenting APIs
- Use **second person** ("you") to address developers directly
- Keep explanations **focused and concise**
- Start with **working examples**, then explain concepts
- Use **`<Callout>`** for important notes and warnings
- Include **complete, runnable code examples** that match actual APIs
- Structure with **`<Steps>`** for sequential instructions
- Use **`<Tabs>`** for package manager commands (with `groupId="package-manager"`)
- Add **`<TypeTable>`** for API reference documentation
- Show **file structures** with `<Files>`, `<Folder>`, `<File>` components
- **Never assume**: If unsure about an API, search and read the implementation

#### 📝 Example Structure:
```mdx
---
title: "Feature Name"
description: "Brief description of what this feature does"
---

## Introduction

What problem does this solve? One paragraph maximum.

<Callout type="info">
  Important context or prerequisite.
</Callout>

## Installation

<Tabs items={['npm', 'pnpm', 'yarn', 'bun']} groupId="package-manager">
  <Tab value="npm">
    ```bash
    npm install @igniter-js/package
    ```
  </Tab>
  <!-- Other package managers -->
</Tabs>

## Quick Start

<Steps>
  <Step>
    ### Install and Configure
    Complete code example that works.
  </Step>
  
  <Step>
    ### Use the Feature
    Another working example.
  </Step>
</Steps>

## API Reference

<TypeTable type={{
  propertyName: {
    type: 'string',
    description: 'What this does',
    required: true
  }
}} />
```

---

### 📗 Blog Posts: Writing Style

**Tone:** Engaging, conversational, and inspiring  
**Voice:** A developer sharing insights and experiences

#### ✅ Style Rules:
- Use **first person** ("I", "we") or **second person** ("you")
- Tell a **story** - what's the journey or insight?
- Include **real-world examples** and use cases
- Use **visual elements** (images, diagrams, demos)
- Add **`<Callout>`** for key takeaways
- Include **working code examples** with context
- End with **clear next steps** or call-to-action
- Use **tags** to categorize content

#### 📝 Example Structure:
```mdx
---
title: "Introducing Real-Time Features in Igniter.js"
description: "How we built SSE-based real-time updates that just work"
tags: ["announcement", "feature", "real-time"]
cover: "https://example.com/cover.jpg"
---

## The Challenge

Developers told us real-time features were too complex...

## Our Solution

We built SSE-based real-time updates that work out of the box:

```typescript
// One line of code for real-time
export const posts = igniter.query({
  stream: true, // That's it!
  handler: async () => { /* ... */ }
});
```

<Callout type="success" title="Key Benefit">
  Real-time updates with zero configuration.
</Callout>

## How It Works

Deep dive into the implementation...

## Getting Started

<Steps>
  <Step>
    ### Install the Package
    Installation code
  </Step>
</Steps>

## What's Next

We're working on WebSocket support, GraphQL subscriptions...
```

---

### 📙 Templates: Writing Style

**Tone:** Practical, clear, and motivating  
**Voice:** A guide showing what's possible

#### ✅ Style Rules:
- Focus on **what's included** out-of-the-box
- Use **`<Files>`** to visualize project structure
- Add **`<Steps>`** for quick start instructions
- Show **key features** with code examples
- Include **deployment instructions**
- Use **`<Callout>`** for important setup notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipebarcelospro/igniter-js](https://github.com/felipebarcelospro/igniter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
