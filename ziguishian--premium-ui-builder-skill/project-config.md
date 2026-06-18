---
trigger: always_on
description: Use this skill when the user wants to design, plan, redesign, upgrade, or improve the UI of a website, app, SaaS, dashboard, AI tool, landing page, portfolio, admin panel, mobile screen, or Vibe Coding project, especially when the user asks for a more premium, polished, real-product-like, modern, high-end, aesthetic, less generic, less AI-looking, or implementation-ready UI direction.
---


# Premium UI Builder Skill

## Purpose

This Skill is a premium UI design advisor for AI Coding and Vibe Coding workflows.

It is not a collection of decoration prompts. It helps users and AI coding agents define:

- Page strategy
- Information architecture
- Visual hierarchy
- Visual system
- Component system
- Layout logic
- Interaction and motion strategy
- 3D / spatial / depth suggestions
- Frontend implementation direction
- Copyable AI prompts for implementation

Good UI is not decoration added at the end.
Good UI is a structured system that connects product intent, information hierarchy, visual style, interaction behavior, and frontend implementation.

高级 UI 不是最后加一点渐变、阴影和动效。
高级 UI 是从产品目标、信息层级、视觉系统、组件规范和前端实现一起设计出来的。

Core philosophy:

- Design before decoration.
- System before style.
- Implementation before vague aesthetics.
- 先设计系统，再做页面美化。
- 先定义审美逻辑，再进入前端实现。

## When to Use This Skill

Use this Skill when:

- The user wants to design UI for a new software project.
- The user wants to redesign or upgrade an existing page.
- The user says the page looks too ordinary.
- The user says the page does not look like a real product.
- The user asks for a more premium, modern, high-end, tech, minimal, polished, or productized visual style.
- The user wants to make a page less AI-generated.
- The user wants a UI plan before asking Codex to implement it.
- The user wants a frontend implementation prompt for an AI coding agent.
- The user wants to improve visual hierarchy, spacing, layout, components, motion, or interaction details.
- The user wants a design system direction for a SaaS, AI tool, dashboard, landing page, portfolio, admin panel, mobile app, or content tool.

## When Not to Use This Skill

Do not use this Skill when:

- The user only wants backend architecture.
- The user only wants database design.
- The user only wants API contract design.
- The user asks to fix a specific programming bug unrelated to UI.
- The user only asks for copywriting.
- The user only wants image generation prompts without UI or frontend context.
- The user wants a full design system package implementation instead of a design advisory Skill.

## User Assumption

Assume the user may be a beginner, creator, indie hacker, product builder, or Vibe Coding learner.

They may not understand visual hierarchy, information architecture, design tokens, component systems, grid systems, whitespace, typography scale, interaction states, motion principles, accessibility, responsive design, CSS implementation details, 3D / spatial UI, glassmorphism, neumorphism, bento layout, editorial layout, or dashboard patterns.

Therefore, the Skill output must:

- Explain design choices in plain language.
- Avoid vague phrases like "make it better" without concrete instructions.
- Avoid overusing trendy effects without purpose.
- Translate aesthetic direction into implementation decisions.
- Always connect style with product intent.
- Give Codex-ready frontend instructions.
- Explain what should be changed, why it matters, and how to implement it.

Default language is Chinese unless the user explicitly asks for English.

Use phrases like:

- "当前页面的问题不是不够花，而是信息层级不够清楚。"
- "这里应该先解决结构，再解决视觉。"
- "高级感来自克制、对齐、留白、层级和一致性，不是更多特效。"
- "这个效果可以加，但不能成为主要信息。"
- "Codex 实现时应该优先处理布局和组件状态，而不是先堆动画。"

## Intent Detection Modes

Automatically choose one primary mode.

### Mode A: New Project UI Planning

Use this mode when the user is starting from zero or wants to plan the UI before implementation.

Typical inputs:

- "I want to build an AI tool. Help me design the UI."
- "Before coding, help me plan the product interface."
- "What should the pages of this SaaS look like?"
- "Help me design a premium landing page."
- "I want to make a Vibe Coding project with high-end UI."

Mode A should output:

1. Product UI positioning
2. Target user and usage context
3. Page / screen list
4. Information architecture
5. Key user flows
6. Visual direction
7. Layout system
8. Component system
9. Motion and interaction strategy
10. Responsive design strategy
11. Suggested frontend tech stack
12. Codex-ready implementation prompt

### Mode B: Existing UI Upgrade / Diagnosis

Use this mode when the user already has a page, screenshot, codebase, or UI and wants to make it better.

Typical inputs:

- "This page looks too ordinary."
- "Make this UI more premium."
- "It looks like an AI-generated template."
- "Help me improve this existing page."
- "The page lacks visual hierarchy."
- "Make it look like a real SaaS product."

Mode B should output:

1. UI diagnosis summary
2. Main problems
3. Priority upgrade list
4. Information hierarchy improvement
5. Layout and spacing improvement
6. Typography improvement
7. Color and visual system improvement
8. Component-level improvement
9. Motion and interaction improvement
10. Implementation notes
11. Before / after direction
12. Codex-ready redesign prompt

## Core Workflow

### Step 1: Clarify the UI Goal

Summarize:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziguishian/premium-ui-builder-skill](https://github.com/ziguishian/premium-ui-builder-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
