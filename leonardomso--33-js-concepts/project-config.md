---
trigger: always_on
description: This repository is a curated collection of **33 essential JavaScript concepts** that every JavaScript developer should know. It serves as a comprehensive learning resource and study guide for developers at all levels, from beginners to advanced practitioners.
---

# 33 JavaScript Concepts - Project Context

## Overview

This repository is a curated collection of **33 essential JavaScript concepts** that every JavaScript developer should know. It serves as a comprehensive learning resource and study guide for developers at all levels, from beginners to advanced practitioners.

The project was recognized by GitHub as one of the **top open source projects of 2018** and has been translated into 40+ languages by the community.

## Project Purpose

- Help developers master fundamental and advanced JavaScript concepts
- Provide curated resources (articles, videos, books) for each concept
- Serve as a reference guide for interview preparation
- Foster community contributions through translations and resource additions

## Repository Structure

```
33-js-concepts/
├── .claude/                 # Claude configuration
│   ├── CLAUDE.md           # Project context and guidelines
│   └── skills/             # Custom skills for content creation
│       ├── write-concept/  # Skill for writing concept documentation
│       ├── fact-check/     # Skill for verifying technical accuracy
│       ├── seo-review/     # Skill for SEO audits
│       ├── test-writer/    # Skill for generating Vitest tests
│       ├── resource-curator/ # Skill for curating external resources
│       └── concept-workflow/ # Skill for end-to-end concept creation
├── .opencode/               # OpenCode configuration
│   └── skill/              # Custom skills (mirrored from .claude/skills)
│       ├── write-concept/  # Skill for writing concept documentation
│       ├── fact-check/     # Skill for verifying technical accuracy
│       ├── seo-review/     # Skill for SEO audits
│       ├── test-writer/    # Skill for generating Vitest tests
│       ├── resource-curator/ # Skill for curating external resources
│       └── concept-workflow/ # Skill for end-to-end concept creation
├── docs/                    # Mintlify documentation site
│   ├── docs.json           # Mintlify configuration
│   ├── index.mdx           # Homepage
│   ├── introduction.mdx    # Getting started guide
│   ├── contributing.mdx    # Contribution guidelines
│   ├── translations.mdx    # Community translations
│   └── concepts/           # 33 concept pages
│       ├── call-stack.mdx
│       ├── primitive-types.mdx
│       └── ... (all 33 concepts)
├── tests/                   # Vitest test suites
│   └── fundamentals/       # Tests for fundamental concepts (1-6)
│       ├── call-stack/
│       ├── primitive-types/
│       ├── value-reference-types/
│       ├── type-coercion/
│       ├── equality-operators/
│       └── scope-and-closures/
├── vitest.config.js        # Vitest configuration
├── README.md               # Main GitHub README
├── CONTRIBUTING.md         # Guidelines for contributors
├── CODE_OF_CONDUCT.md      # Community standards
├── LICENSE                 # MIT License
├── package.json            # Project metadata
├── opencode.jsonc          # OpenCode AI assistant configuration
└── github-image.png        # Project banner image
```

## The 31 Concepts (32nd and 33rd coming soon)

### Fundamentals (1-6)
1. Primitive Types
2. Value Types and Reference Types
3. Type Coercion (Implicit, Explicit, Nominal, Structuring and Duck Typing)
4. Equality Operators (== vs === vs typeof)
5. Scope & Closures
6. Call Stack

### Functions & Execution (7-8)
7. Event Loop (Message Queue)
8. IIFE, Modules and Namespaces

### Web Platform (9-10)
9. DOM and Layout Trees
10. HTTP & Fetch

### Object-Oriented JS (11-15)
11. Factories and Classes
12. this, call, apply and bind
13. new, Constructor, instanceof and Instances
14. Prototype Inheritance and Prototype Chain
15. Object.create and Object.assign

### Functional Programming (16-19)
16. map, reduce, filter
17. Pure Functions, Side Effects, State Mutation and Event Propagation
18. Higher-Order Functions
19. Recursion

### Async JavaScript (20-22)
20. Collections and Generators
21. Promises
22. async/await

### Advanced Topics (23-31)
23. JavaScript Engines
24. Data Structures
25. Big O Notation (Expensive Operations)
26. Algorithms
27. Inheritance, Polymorphism and Code Reuse
28. Design Patterns
29. Partial Applications, Currying, Compose and Pipe
30. Clean Code

## Content Format

Each concept page in `/docs/concepts/` follows this structure:

### 1. Frontmatter
```mdx
---
title: "Concept Name"
description: "Brief description of the concept"
---
```

### 2. Real-World Analogy
Start with an engaging analogy that makes the concept relatable. Include ASCII art diagrams when helpful.

### 3. Info Box (What You'll Learn)
```mdx
<Info>
**What you'll learn in this guide:**
- Key point 1
- Key point 2
- Key point 3
</Info>
```

### 4. Main Content Sections
- Use clear headings (`##`, `###`) to organize topics
- Include code examples with explanations
- Use Mintlify components (`<AccordionGroup>`, `<Steps>`, `<Tabs>`, etc.)
- Add diagrams and visualizations where helpful

### 5. Related Concepts
```mdx
<CardGroup cols={2}>
  <Card title="Related Concept" icon="icon-name" href="/concepts/concept-slug">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardomso/33-js-concepts](https://github.com/leonardomso/33-js-concepts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
