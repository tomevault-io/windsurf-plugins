---
trigger: always_on
description: Welcome, AI Agent or Developer. This document is the supreme behavioral and architectural contract for working within the **Qahera UI Kit** codebase.
---

# AGENTS.md — Qahera UI Kit Governance & Agent Instructions

Welcome, AI Agent or Developer. This document is the supreme behavioral and architectural contract for working within the **Qahera UI Kit** codebase.

Every AI coding agent (Antigravity, Claude Code, Cursor, Copilot, etc.) operating in this repository **MUST** adhere to the instructions and invariants defined herein without exception.

---

## 1. Identity & Sovereign Ownership
 
* **Owner:** **Alwkala** (Egyptian / Regional Technology Studio).
* **Project Family:** Alwkala UI Kits (`Qahera UI Kit`, `Alex Admin Kit`, `Qena UI Kit`).
* **Ecosystem Relationship:** **Coordinated with & compatible with TidyFactor**, but **NOT** owned by it.
* **Separation of Concerns:**
  * **TidyFactor:** Context, Memory, Governance, Skills, CLI, Agent Workflows.
  * **Qahera:** AI-Native Design System, Component Contracts, Recipes, Multi-Target Renderers, Design Tokens.

### 🏛️ The Core Equation

$$\mathbf{\text{Qahera UI Kit v1.0}} = \mathbf{\text{Design System}} + \mathbf{\text{Registry}} + \mathbf{\text{AI Decision Layer}}$$

> **Rule for Agents:** Qahera is **NOT** just a "Component Library + Compiler". The compiler is a frozen, background build tool. The product is an **AI-Native UI Kit** combining a systematic design language, an authoritative canonical registry, and an explicit AI decision layer.

---

## 2. The 15 Non-Negotiable Architectural Invariants

Agents must verify every change against these 15 invariants before proposing or committing code:

1. **YAML is Authoritative:** The canonical source of truth for all tokens, contracts, and recipes is strictly YAML. JSON is generated for machine interchange only.
2. **Generated Files are Never the Source of Truth:** Never patch files inside `renderers/`, `dist/`, or `generated/` to fix a design bug. Fix the `contract/`, `token/`, or `recipe/` source, then regenerate.
3. **Tokens Precede Styling:** Hardcoded hex values, pixel dimensions, raw transitions, or arbitrary inline styles are strictly prohibited. Every visual attribute must trace back to `--qhr-*` CSS custom properties.
4. **Contracts Precede Implementations:** No component or variant may be rendered or implemented without first having a documented contract definition in `contracts/`.
5. **RTL is Infrastructure, Not a Theme:** RTL/LTR parity is built into the core via logical CSS properties (`margin-inline-start`, `padding-inline-end`, `inset-inline-start`). Never create separate `ButtonLTR` and `ButtonRTL` components.
6. **Arabic Typography Discipline:** The primary canonical fonts are **Alexandria** (headings, display, brand identity) and **Cairo** (body copy, UI labels, form controls) via `--qhr-font-family-primary`, `--qhr-font-heading`, and `--qhr-font-body`. Supported secondary alternatives include **El Messiri** (for heritage display), **Tajawal** (alternative body), and **Plus Jakarta Sans / Inter** (Latin). The font **Amiri** is strictly prohibited for UI components.
7. **Heritage Stays in Templates:** Egyptian, Cairo, and Islamic cultural motifs belong to downstream templates, boilerplates, and themes. The core design system contract remains neutral, modern, and universally portable.
8. **Renderer Parity with Zero Vocabulary Drift:** Renderers may vary in binding syntax (e.g. `data-variant="primary"` in HTML vs `$variant = 'primary'` in PHP vs `variant="primary"` in React), but must NEVER alter the semantic prop names, values, or accessibility contracts.
9. **Source Ownership over Runtime Lock-in:** Consuming projects copy and own their component source code (shadcn-style). Do not enforce unnecessary runtime dependencies or package lock-in.
10. **AI Metadata is a First-Class Requirement:** Every component recipe must expose explicit AI metadata: WHAT (purpose), WHEN (use cases), HOW (slots & props), and WHY NOT (anti-patterns & alternatives).
11. **Icons are Canonical Architecture, Never Emoji (`QAHERA-VISUAL-001`):** Emoji (e.g. 🗑️, ✕, 🚀, ❤️, ☀️, 🌙, 🏛️) and arbitrary Unicode symbols are strictly prohibited anywhere in UI components, headers, showcases, or previews. All iconography MUST reference the authoritative `icons/registry.yaml` semantic catalog (`svg_path` with 24x24 viewBox, stroke-width 2) or clean inline SVGs.
12. **Modular Stylesheet Architecture & RSC 0kb Boundary:** Component styles must never be compiled into monolithic manual files. Every component owns its atomic stylesheet in `renderers/html/native/components/*.css`, while `components.css` remains an `@import` manifest and `dist/qahera.css` is emitted via `cli/build-css.js`. React components preserve a 0kb client footprint via pure Server Components, reserving `'use client'` strictly for terminal interactive leaves.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alwkala/Qahera-UI-Kit](https://github.com/alwkala/Qahera-UI-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
