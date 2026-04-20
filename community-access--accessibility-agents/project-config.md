---
trigger: always_on
description: This workspace enforces WCAG AA accessibility standards for all web UI code.
---

## Accessibility-First Development

This workspace enforces WCAG AA accessibility standards for all web UI code.

### Repository Writing Policy: No Emoji

Repository-wide hard rule: do not add emoji characters in generated, edited, or reviewed content.

- Applies to release notes, changelog entries, docs, prompts, instructions, agent files, issue/PR text, and chat-generated copy intended for repository publication.
- Replace emoji with plain text labels.
- If existing content contains emoji and you are touching that content, remove emoji as part of the update.

### Mandatory Accessibility Check

Before writing or modifying any web UI code - including HTML, JSX, CSS, React components, Tailwind classes, web pages, forms, modals, or any user-facing web content - you MUST:

1. Consider which accessibility specialist agents are needed for the task
2. Apply the relevant specialist knowledge before generating code
3. Verify the output against the appropriate checklists

**Automatic trigger detection:** If a user prompt involves creating, editing, or reviewing any file matching `*.html`, `*.jsx`, `*.tsx`, `*.vue`, `*.svelte`, `*.astro`, or `*.css` - or if the prompt describes building UI components, pages, forms, or visual elements - treat it as a web UI task and apply the Decision Matrix below to determine which specialists are needed. Do not wait for the user to explicitly request accessibility review.

### Available Specialist Agents

Select these agents from the agents dropdown in Copilot Chat, or type `/agents` to browse:

| Agent | When to Use |
|-------|------------|
| Accessibility Lead | Any UI task - coordinates all specialists and runs final review |
| ARIA Specialist | Interactive components, custom widgets, ARIA usage |
| Modal Specialist | Dialogs, drawers, popovers, overlays |
| Contrast Master | Colors, themes, CSS styling, visual design |
| Keyboard Navigator | Tab order, focus management, keyboard interaction |
| Live Region Controller | Dynamic content updates, toasts, loading states |
| Forms Specialist | Forms, inputs, validation, error handling, multi-step wizards |
| Alt Text & Headings | Images, alt text, SVGs, heading structure, page titles, landmarks |
| Tables Specialist | Data tables, sortable tables, grids, comparison tables, pricing tables |
| Link Checker | Ambiguous link text, "click here"/"read more" detection, link purpose |
| Cognitive Accessibility | WCAG 2.2 cognitive SC, COGA guidance, plain language, authentication UX |
| Mobile Accessibility | React Native, Expo, iOS, Android - touch targets, screen reader compatibility |
| Design System Auditor | Color token contrast, focus ring tokens, spacing tokens, Tailwind/MUI/Chakra/shadcn |
| Markdown Accessibility | Markdown document accessibility - links, alt text, headings, tables, emoji, mermaid diagrams, dashes, anchors |
| Web Accessibility Wizard | Full guided web accessibility audit with step-by-step walkthrough |
| Document Accessibility Wizard | Document accessibility audit for .docx, .xlsx, .pptx, .pdf - single files, folders, recursive scanning, delta scanning, severity scoring, remediation tracking, compliance export (VPAT/ACR), CSV export with help links, CI/CD integration |
| Testing Coach | Screen reader testing, keyboard testing, automated testing guidance |
| WCAG Guide | WCAG 2.2 criteria explanations, conformance levels, what changed |
| Developer Hub | Python, wxPython, desktop app development - routes to specialist agents, scaffolds, debugs, reviews, builds |
| Python Specialist | Python debugging, packaging (PyInstaller/Nuitka/cx_Freeze), testing, type checking, async, optimization |
| wxPython Specialist | wxPython GUI - sizer layouts, event handling, AUI, custom controls, threading, desktop accessibility |
| Desktop Accessibility Specialist | Desktop application accessibility - platform APIs (UI Automation, MSAA/IAccessible2, NSAccessibility), accessible control patterns, screen reader Name/Role/Value/State, focus management, high contrast, and custom widget accessibility for Windows and macOS desktop applications |
| Desktop A11y Testing Coach | Desktop accessibility testing - testing with NVDA, JAWS, Narrator, and VoiceOver screen readers, Accessibility Insights for Windows, automated UIA testing, keyboard-only testing flows, high contrast verification, and creating desktop accessibility test plans |
| Accessibility Tool Builder | Building accessibility scanning tools, rule engines, document parsers, report generators, and audit automation. WCAG criterion mapping, severity scoring algorithms, CLI/GUI scanner architecture, and CI/CD integration for accessibility tooling |
| Media Accessibility | Video and audio accessibility - captions, audio descriptions, transcripts, media player controls, WCAG 1.2.x compliance |
| Email Accessibility | HTML email accessibility under email client rendering constraints - table-based layout, inline styles, image fallbacks, screen reader compatibility |
| Data Visualization Accessibility | Chart, graph, and dashboard accessibility - SVG ARIA, data table alternatives, color-safe palettes, keyboard interaction |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Community-Access/accessibility-agents](https://github.com/Community-Access/accessibility-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
