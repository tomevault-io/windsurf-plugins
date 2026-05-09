---
trigger: always_on
description: **Template Version**: 1.0.1
---

# Leptos 0.8 CSR Project - Development Guide

**Template Version**: 1.0.1
**Last Updated**: 2025-12-25
**Stack**: Leptos 0.8 (CSR) + Trunk + TailwindCSS 3.4.18 + DaisyUI 4.12.24 + Iconify Lucide

> ⚠️ **IMPORTANT**: This project uses **DaisyUI 4.12.24** (compatible with Tailwind CSS 3.4.18).
> DaisyUI 5.x requires Tailwind CSS 4 and is **NOT compatible** with this setup.

> **Note**: This is a general template for Leptos 0.8 CSR projects. Customize the "Project-Specific Context" section at the end for your specific application.

---

## Overview

This is a **Leptos 0.8 Client-Side Rendered (CSR)** web application built with modern Rust web technologies. This document serves as the architectural foundation and development guide for the project.

### Technology Stack

#### Core Framework
- **Leptos 0.8**: Reactive web framework for Rust
  - Mode: CSR (Client-Side Rendering)
  - Compilation target: `wasm32-unknown-unknown`
  - Requires: Rust nightly toolchain

#### Build System
- **Trunk**: WASM web application bundler
  - Handles WASM compilation
  - Manages assets and index.html
  - Development server with hot reload

#### Styling
- **TailwindCSS 3.4.18**: Utility-first CSS framework
  - JIT (Just-In-Time) compiler
  - Scans `*.rs` files for class names
  - Mobile-first responsive design

- **DaisyUI 4.12.24**: Component library for Tailwind
  - Semantic component classes
  - Theme system (light/dark)
  - WCAG AA accessible
  - ⚠️ **CRITICAL**: Use version 4.x, NOT 5.x (DaisyUI 5 requires Tailwind CSS 4)

- **Iconify Lucide**: Icon system
  - Format: `icon-[lucide--icon-name]`
  - Dynamic icon loading via `@iconify/tailwind`
  - Vector icons with CSS sizing/coloring

#### Development Dependencies
- **Node.js/npm**: For CSS tooling (Tailwind, DaisyUI)
- **cargo-leptos** (optional): Additional build utilities
- **wasm-bindgen**: JavaScript interop

---

## Project Structure

### Recommended Directory Layout

```
project-root/
├── .claude/                          # Claude Code tooling (optional)
│   ├── skills/                      # Custom skills for component generation
│   ├── agents/                      # Sub-agents for specialized tasks
│   └── hooks/                       # Git hooks (pre-commit, pre-push)
│
├── src/
│   ├── lib.rs                       # App entry point, router setup
│   │
│   ├── pages/                       # Top-level pages/routes
│   │   ├── home.rs
│   │   ├── about.rs
│   │   └── mod.rs
│   │
│   ├── ui/                          # Dumb/Presentational components
│   │   ├── atoms/                  # Single-purpose components
│   │   │   ├── button.rs           # Button, Badge, Input, Icon
│   │   │   └── mod.rs
│   │   ├── molecules/              # Compositions of atoms
│   │   │   ├── card.rs             # Card, FormField, SearchBar
│   │   │   └── mod.rs
│   │   ├── organisms/              # Complex compositions
│   │   │   ├── navbar.rs           # Navbar, Footer, Sidebar
│   │   │   └── mod.rs
│   │   └── mod.rs
│   │
│   ├── features/                    # Feature modules (domain-based)
│   │   ├── {feature_name}/
│   │   │   ├── components/         # Smart components for this feature
│   │   │   │   ├── feature_form.rs
│   │   │   │   └── mod.rs
│   │   │   ├── context.rs          # Feature-specific state (Context API)
│   │   │   ├── api.rs              # Backend integration
│   │   │   └── mod.rs
│   │   └── mod.rs
│   │
│   ├── context/                     # Global contexts (optional)
│   │   ├── theme.rs                # Theme context
│   │   ├── auth.rs                 # Authentication context
│   │   └── mod.rs
│   │
│   └── utils/                       # Shared utilities (optional)
│       ├── format.rs
│       └── mod.rs
│
├── public/                          # Static assets
│   └── assets/
│
├── index.html                       # HTML template (used by Trunk)
├── Cargo.toml                       # Rust dependencies
├── package.json                     # Node.js dependencies (Tailwind, DaisyUI)
├── tailwind.config.js               # Tailwind configuration
├── rust-toolchain.toml              # Rust nightly version
└── Trunk.toml                       # Trunk configuration
```

---

## Architecture Principles

### 1. Bottom-Up Design

Build from simple to complex:

```
Atoms (ui/atoms/)
  ↓
Molecules (ui/molecules/)
  ↓
Organisms (ui/organisms/)
  ↓
Features (features/)
  ↓
Pages (pages/)
```

**Example**:
- **Atom**: `Button` component
- **Molecule**: `Card` component (uses Button)
- **Organism**: `Navbar` component (uses Card, Button)
- **Feature**: `Auth` feature (uses Navbar, custom forms)
- **Page**: `HomePage` (composes features and organisms)

**Rule**: Never start with page-level components. Always identify and build the atomic pieces first.

### 2. DRY (Don't Repeat Yourself)

- Extract repeated patterns into reusable components
- Use variant enums for type-safe styling
- Create shared utilities for common operations
- Avoid copy-pasting code blocks
- If you write the same code pattern 3+ times, extract it

**Bad**:
```rust
// In component A
<button class="btn btn-primary">

// In component B
<button class="btn btn-primary">

// Repeated 10+ times...
```

**Good**:
```rust
// Create Button component in ui/atoms/
<Button variant=ButtonVariant::Primary>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [automataIA/dashboard-studio-rs](https://github.com/automataIA/dashboard-studio-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
