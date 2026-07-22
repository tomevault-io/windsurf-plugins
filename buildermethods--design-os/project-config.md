---
trigger: always_on
description: Design OS is a **product planning and design tool** that helps users define their product vision, sketch out their data shape, design their UI, and prepare export packages for implementation in a separate codebase.
---

# Agent Directives for Design OS

Design OS is a **product planning and design tool** that helps users define their product vision, sketch out their data shape, design their UI, and prepare export packages for implementation in a separate codebase.

> **Important**: Design OS is a planning tool, not the end product codebase. The screen designs and components generated here are meant to be exported and integrated into your actual product's codebase.

---

## Understanding Design OS Context

When working in Design OS, be aware of two distinct contexts:

### 1. Design OS Application
The React application that displays and manages planning files. When modifying the Design OS UI itself:
- Files live in `src/` (components, pages, utilities)
- Uses the Design OS design system (stone palette, DM Sans, etc.)
- Provides the interface for viewing specs, screen designs, exports, etc.

### 2. Product Design (Screen Designs & Exports)
The product you're planning and designing. When creating screen designs and exports:
- Screen design components live in `src/sections/[section-name]/` and `src/shell/`
- Product definition files live in `product/`
- Exports are packaged to `product-plan/` for integration into a separate codebase
- Follow the design requirements specified in each section's spec

---

## Getting Started — The Planning Flow

Design OS follows a structured planning sequence:

### 1. Product Vision (`/product-vision`)
Define your product overview, roadmap sections, and data shape — all in one conversational flow. After answering clarifying questions, all three files are generated automatically.
**Output:** `product/product-overview.md`, `product/product-roadmap.md`, `product/data-shape/data-shape.md`

Use `/product-roadmap`, `/data-shape` individually to update those files after initial creation.

### 2. Design System (`/design-tokens`)
Choose your color palette (from Tailwind) and typography (from Google Fonts). These tokens are applied to all screen designs.
**Output:** `product/design-system/colors.json`, `product/design-system/typography.json`

### 3. Application Shell (`/design-shell`)
Design the persistent navigation and layout that wraps all sections.
**Output:** `product/shell/spec.md`, `src/shell/components/`

### 4. For Each Section:
- `/shape-section` — Define the specification and generate sample data + types
- `/sample-data` — Update sample data and types (if already created)
- `/design-screen` — Create screen designs
- `/screenshot-design` — Capture screenshots

### 5. Export (`/export-product`)
Generate the complete export package with all components, types, and handoff documentation.
**Output:** `product-plan/`

---

## File Structure

```
product/                           # Product definition (portable)
├── product-overview.md            # Product description, problems/solutions, features
├── product-roadmap.md             # List of sections with titles and descriptions
│
├── data-shape/                    # Product data shape
│   └── data-shape.md              # Entity names, descriptions, and relationships
│
├── design-system/                 # Design tokens
│   ├── colors.json                # { primary, secondary, neutral }
│   └── typography.json            # { heading, body, mono }
│
├── shell/                         # Application shell
│   └── spec.md                    # Shell specification
│
└── sections/
    └── [section-name]/
        ├── spec.md                # Section specification
        ├── data.json              # Sample data for screen designs
        ├── types.ts               # TypeScript interfaces
        └── *.png                  # Screenshots

src/
├── shell/                         # Shell design components
│   ├── components/
│   │   ├── AppShell.tsx
│   │   ├── MainNav.tsx
│   │   ├── UserMenu.tsx
│   │   └── index.ts
│   └── ShellPreview.tsx
│
└── sections/
    └── [section-name]/
        ├── components/            # Exportable components
        │   ├── [Component].tsx
        │   └── index.ts
        └── [ViewName].tsx         # Preview wrapper

product-plan/                      # Export package (generated)
├── README.md                      # Quick start guide
├── product-overview.md            # Product summary
├── prompts/                       # Ready-to-use prompts for coding agents
│   ├── one-shot-prompt.md         # Prompt for full implementation
│   └── section-prompt.md          # Prompt template for incremental
├── instructions/                  # Implementation instructions
│   ├── one-shot-instructions.md   # All milestones combined
│   └── incremental/               # Milestone-by-milestone instructions
│       ├── 01-shell.md
│       └── [NN]-[section-id].md   # Section-specific instructions
├── design-system/                 # Tokens, colors, fonts
├── data-shapes/                   # UI data contracts (types components expect)
├── shell/                         # Shell components
└── sections/                      # Section components (with tests.md each)
```

---

## Design Requirements

When creating screen designs, follow these guidelines:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buildermethods/design-os](https://github.com/buildermethods/design-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
