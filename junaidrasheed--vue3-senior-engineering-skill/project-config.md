---
trigger: always_on
description: |
---


# Vue 3 Senior Engineering Skill

A comprehensive skill for generating production-grade Vue 3 code following architectural best practices, team conventions, and senior-level engineering standards.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [When to Use This Skill](#when-to-use-this-skill)
3. [Workflow: Planning Before Code](#workflow-planning-before-code)
4. [Component Architecture](#component-architecture)
5. [State Management & Pinia](#state-management--pinia)
6. [Composables & Reusable Logic](#composables--reusable-logic)
7. [Testing Strategy](#testing-strategy)
8. [HTTP & API Integration](#http--api-integration)
9. [Styling with Tailwind CSS](#styling-with-tailwind-css)
10. [Nuxt-Specific Patterns](#nuxt-specific-patterns)
11. [Code Quality Standards](#code-quality-standards)
12. [Performance & Optimization](#performance--optimization)
13. [Architecture Decision Records](#architecture-decision-records)
14. [Appendix: Quick Reference](#appendix-quick-reference)

---

## Core Principles

This skill enforces a unified set of architectural and coding standards designed for scalability, maintainability, and long-term team success:

1. **Production-Grade First**: All code is optimized, structured, and test-ready from day one
2. **Composition API Always**: Vue 3 Composition API is the standard; Options API only for legacy code
3. **Separation of Concerns**: Clear boundaries between UI logic, business logic, and data access
4. **Type Safety Enforced**: Strict TypeScript mode, no `any` types, explicit return types everywhere
5. **Pragmatism Over Dogma**: Architecture serves the team and users, not rules for their own sake
6. **Senior-Level Collaboration**: Ask clarifying questions, offer alternatives with rationale, guide decisions
7. **Performance Conscious**: Minimize re-renders, optimize reactivity, reduce bundle bloat
8. **Testability Built-In**: Design for testing; include factory functions and test patterns by default
9. **Documentation & Clarity**: Code is readable, maintainable, and well-documented
10. **Consistency Across Teams**: Enforce naming conventions, file organization, and patterns uniformly

---

## When to Use This Skill

### Explicit Triggers (Always Use)

- **Writing new Vue 3 components** (feature components, base components, containers)
- **Refactoring existing Vue code** (Options API → Composition API, improving structure)
- **Creating or updating composables** (reusable logic, hooks)
- **Building/updating Pinia stores** (state management, actions, getters)
- **Creating Vue plugins** (custom functionality, global features)
- **Performance optimization** (reducing re-renders, optimizing reactivity, code splitting)
- **Frontend architecture discussions** (folder structure, data flow, patterns)
- **Debugging Vue-specific patterns** (reactivity issues, lifecycle, watchers)
- **Converting Options API to Composition API** (modernizing legacy code)
- **Optimizing rendering, reactivity, or state** handling
- **Code reviews of Vue code** (enforcing standards, suggesting improvements)
- **Building HTTP clients and API services** (fetch integration, request/response transformation)
- **Setting up Nuxt applications** (SSR, auto-imports, middleware, server boundaries)
- **Writing validation schemas** (Zod integration, type safety)
- **Creating utility functions** (one function per file, global imports)
- **Configuring Tailwind CSS** (design systems, utility composition)

### Input Scenarios

The skill accepts:
- Existing `.vue` files (components)
- `.ts` / `.js` files (composables, utilities, services)
- Pinia store files
- Plugin files
- Feature requirements (description of desired behavior)
- UI requirements (design specs, layout needs)
- Refactor requests (code smell, architectural issues)
- Bug reports (reactivity, performance, behavior issues)
- Architecture discussions (structure, patterns, data flow)
- Performance concerns (slow renders, bundle size, memory)
- Code snippets and partial implementations
- Configuration files (Tailwind, Vite, Nuxt)

---

## Workflow: Planning Before Code

### Step 1: Clarification & Planning (Always Start Here)

**Never jump straight to code generation.** Always plan first.

Ask clarifying questions to understand:
1. **What is the feature or task?** (Clear description, context, requirements)
2. **What's the current state?** (Existing code, architecture, constraints)
3. **What problem are we solving?** (User impact, business value, technical goal)
4. **Are there existing patterns?** (In the codebase, team standards)
5. **Scale & complexity:** (Single component, feature, refactor, architecture?)
6. **Constraints:** (Performance, compatibility, accessibility, security)
7. **Integration points:** (API, stores, other components, routing)

**Output: Clear Plan Before Code**

Once you understand the context, propose a plan (not code):

```markdown
## Plan

### Overview
[Clear description of what we're building]

### Structure
- [Component hierarchy or file organization]
- [State management approach]
- [API integration points]

### Key Decisions
- [Architecture patterns]
- [Composable/store breakdown]
- [Performance considerations]

### Implementation Steps
1. [Create base structure]
2. [Implement core logic]
3. [Add tests]
4. [Optimize]

### Questions Before We Proceed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junaidrasheed/vue3-senior-engineering-skill](https://github.com/junaidrasheed/vue3-senior-engineering-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
