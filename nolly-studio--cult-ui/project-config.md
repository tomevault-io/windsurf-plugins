---
trigger: always_on
description: Handles user interaction:
---

# Components.build Specification

**Version 1.0.0**  
components.build  
January 2026

> **Note:**  
> This document is mainly for agents and LLMs to follow when maintaining,  
> generating, or refactoring UI component libraries. Humans  
> may also find it useful, but guidance here is optimized for automation  
> and consistency by AI-assisted workflows.

---

## Abstract

Comprehensive guidelines for building modern, composable, and accessible UI components. This specification provides patterns and best practices for creating React components that are composable, accessible, customizable, and well-documented.

---

## Table of Contents

1. [Overview](#1-overview) — **MEDIUM**
   - 1.1 [Components.build Overview](#11-componentsbuild-overview)
2. [Principles](#2-principles) — **HIGH**
   - 2.1 [Core Component Principles](#21-core-component-principles)
3. [Definitions](#3-definitions) — **MEDIUM**
   - 3.1 [Component Artifact Definitions](#31-component-artifact-definitions)
4. [Composition](#4-composition) — **HIGH**
   - 4.1 [Component Composition](#41-component-composition)
5. [Accessibility](#5-accessibility) — **CRITICAL**
   - 5.1 [Accessibility Guidelines](#51-accessibility-guidelines)
6. [State](#6-state) — **HIGH**
   - 6.1 [State Management Patterns](#61-state-management-patterns)
7. [Types](#7-types) — **HIGH**
   - 7.1 [Component Types](#71-component-types)
8. [Polymorphism](#8-polymorphism) — **MEDIUM**
   - 8.1 [Polymorphism Patterns](#81-polymorphism-patterns)
9. [As-Child](#9-as-child) — **MEDIUM**
   - 9.1 [asChild Pattern](#91-aschild-pattern)
10. [Data Attributes](#10-data-attributes) — **LOW**
   - 10.1 [Data Attributes for Styling](#101-data-attributes-for-styling)
11. [Styling](#11-styling) — **HIGH**
   - 11.1 [Component Styling with Tailwind CSS](#111-component-styling-with-tailwind-css)
12. [Design Tokens](#12-design-tokens) — **MEDIUM**
   - 12.1 [Design Tokens](#121-design-tokens)
13. [Documentation](#13-documentation) — **MEDIUM**
   - 13.1 [Component Documentation](#131-component-documentation)
14. [Registry](#14-registry) — **LOW**
   - 14.1 [Component Registries](#141-component-registries)
15. [NPM](#15-npm) — **LOW**
   - 15.1 [Publishing to NPM](#151-publishing-to-npm)
16. [Marketplaces](#16-marketplaces) — **LOW**
   - 16.1 [Component Marketplaces](#161-component-marketplaces)

---

## 1. Overview

**Impact: MEDIUM**

Specification scope, goals, and philosophy. Introduction to the
components.build standard for building modern UI components.

### 1.1 Components.build Overview

**Impact: MEDIUM (Foundation for understanding the specification)**

The components.build specification is an open-source standard for building modern, composable, and accessible UI components. It provides high-level guidelines, best practices, and common terminology for designing UI components that integrate smoothly into any codebase.

**What This Specification Is:**

This spec is **not**:

- A tutorial or course on React

- A promotion for any specific component library or registry

- A replacement for framework documentation

This spec **is**:

- A set of high-level guidelines and best practices

- A common terminology for designing UI components

- A standard for ensuring components meet modern expectations

- A framework for creating components that integrate smoothly across projects

**Who This Is For:**

This specification is written for:

- **Open-source maintainers** building and distributing component libraries

- **Senior front-end engineers** designing component APIs and design systems

- **Developers** familiar with JavaScript/TypeScript and React

**Framework Scope:**

While examples use React (with JSX/TSX) for concreteness, the fundamental concepts apply to other frameworks (Vue, Svelte, Angular). The philosophy is **framework-agnostic**.

**Core Goals:**

The specification aims to help developers create components that are:

1. **Composable** - Components combine and nest to create complex UIs

2. **Accessible** - Usable by everyone, including users with disabilities

3. **Easy to adopt** - Integrate smoothly into any codebase

4. **Consistent** - Follow modern expectations and patterns

5. **Well-documented** - Clear guidelines and terminology

**Key Philosophy:**

- **Composition over configuration** - Build flexible, composable APIs

- **Accessibility by default** - Not an afterthought, but a requirement

- **Developer experience** - Components should be easy to understand, customize, and integrate

- **Transparency** - Source code should be inspectable and modifiable

- **Standards alignment** - Follow web standards and modern best practices

**Example:**

The following examples illustrate the difference between components that don't follow the specification and those that do:

**Incorrect:**

```tsx
// Hard-coded styles, no accessibility, not composable
function Button() {
  return (
    <button style={{ backgroundColor: '#007bff', color: 'white', padding: '10px' }}>
      Click me
    </button>
  );
}
```

**Correct:**

```tsx
// Composable, accessible, customizable
import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'outline';
}

export function Button({ 
  className, 
  variant = 'default', 
  children, 
  ...props 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nolly-studio/cult-ui](https://github.com/nolly-studio/cult-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
