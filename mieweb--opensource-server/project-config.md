---
trigger: always_on
description: - **Never duplicate code**: If you find yourself copying code, extract it into a reusable function
---

## Code Quality Principles

### 🎯 DRY (Don't Repeat Yourself)
- **Never duplicate code**: If you find yourself copying code, extract it into a reusable function
- **Single source of truth**: Each piece of knowledge should have one authoritative representation
- **Refactor mercilessly**: When you see duplication, eliminate it immediately
- **Shared utilities**: Common patterns should be abstracted into utility functions

### 💋 KISS (Keep It Simple, Stupid)
- **Simple solutions**: Prefer the simplest solution that works
- **Avoid over-engineering**: Don't add complexity for hypothetical future needs
- **Clear naming**: Functions and variables should be self-documenting
- **Small functions**: Break down complex functions into smaller, focused ones
- **Readable code**: Code should be obvious to understand at first glance

### 🧹 Folder Philosophy
- **Clear purpose**: Every folder should have a main thing that anchors its contents.
- **No junk drawers**: Don’t leave loose files without context or explanation.
- **Explain relationships**: If it’s not elegantly obvious how files fit together, add a README or note.
- **Immediate clarity**: Opening a folder should make its organizing principle clear at a glance.

### ⚛️ React Components
- **One component per file**: Every React component lives in its own file named after the component (e.g. `MultiSelect.tsx`). No inline helper components inside another component's file — extract them.

### 🔄 Refactoring Guidelines
- **Continuous improvement**: Refactor as you work, not as a separate task
- **Safe refactoring**: Always run tests before and after refactoring
- **Incremental changes**: Make small, safe changes rather than large rewrites
- **Preserve behavior**: Refactoring should not change external behavior
- **Code reviews**: All refactoring should be reviewed for correctness

### ⚰️ Dead Code Management
- **Immediate removal**: Delete unused code immediately when identified
- **Historical preservation**: Move significant dead code to `.attic/` directory with context
- **Documentation**: Include comments explaining why code was moved to attic
- **Regular cleanup**: Review and clean attic directory periodically
- **No accumulation**: Don't let dead code accumulate in active codebase

## Accessibility (ARIA Labeling)

### 🎯 Interactive Elements
- **All interactive elements** (buttons, links, forms, dialogs) must include appropriate ARIA roles and labels
- **Use ARIA attributes**: Implement aria-label, aria-labelledby, and aria-describedby to provide clear, descriptive information for screen readers
- **Semantic HTML**: Use semantic HTML wherever possible to enhance accessibility

### 📢 Dynamic Content
- **Announce updates**: Ensure all dynamic content updates (modals, alerts, notifications) are announced to assistive technologies using aria-live regions
- **Maintain tab order**: Maintain logical tab order and keyboard navigation for all features
- **Visible focus**: Provide visible focus indicators for all interactive elements

## Internationalization (I18N)

### 🌍 Text and Language Support
- **Externalize text**: All user-facing text must be externalized for translation
- **Multiple languages**: Support multiple languages, including right-to-left (RTL) languages such as Arabic and Hebrew
- **Language selector**: Provide a language selector for users to choose their preferred language

### 🕐 Localization
- **Format localization**: Ensure date, time, number, and currency formats are localized based on user settings
- **UI compatibility**: Test UI layouts for text expansion and RTL compatibility
- **Unicode support**: Use Unicode throughout to support international character sets

## Documentation Preferences

### Writing Style
- **Terse and practical**: Every sentence must convey actionable or necessary information. Remove filler, marketing language, and motivational text.
- **No fluff sections**: Do not include "What is X?", "Why use X?", "Benefits of X", "Key Features", or "Next Steps" sections. Readers already chose the tool — tell them how to use it.
- **No troubleshooting sections**: Do not add generic troubleshooting or "Common Issues" sections to docs pages.
- **Minimal repetition**: State a fact once. Cross-reference instead of restating. If two pages cover overlapping topics, consolidate into one and link.
- **Lead with commands, config, and examples**: Tables, code blocks, CLI commands, and configuration snippets are preferred over prose explanations.
- **Short introductions**: A page introduction should be one or two sentences stating what the page covers, then immediately begin the content.

### Diagrams and Visual Documentation
- **Always use Mermaid diagrams** instead of ASCII art for workflow diagrams, architecture diagrams, and flowcharts
- Use appropriate Mermaid diagram types:
  - `graph TB` or `graph LR` for workflow architectures 
  - `flowchart TD` for process flows
  - `sequenceDiagram` for API interactions
  - `gitgraph` for branch/release strategies
- Include styling with `classDef` for better visual hierarchy

### Documentation Standards
- Keep documentation DRY — reference other docs instead of duplicating content
- Use clear cross-references between related documentation files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mieweb/opensource-server](https://github.com/mieweb/opensource-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
