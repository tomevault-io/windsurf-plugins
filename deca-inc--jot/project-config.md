---
trigger: always_on
description: This document provides essential context for AI assistants (like Claude) working on this codebase. It covers architecture, coding guidelines, and product fundamentals.
---

# Claude Development Guide

This document provides essential context for AI assistants (like Claude) working on this codebase. It covers architecture, coding guidelines, and product fundamentals.

## Table of Contents

- [Product Fundamentals](#product-fundamentals)
- [Architecture](#architecture)
- [Deployment](#deployment)
- [Coding Guidelines](#coding-guidelines)
- [Design System](#design-system)
- [Data Model](#data-model)
- [Security & Privacy](#security--privacy)

---

## Product Fundamentals

### Core Principles

#### 1. Writing Experience is the Most Important Thing

The typing and writing experience must be exceptional—competitive with the best writing apps (iA Writer, Ulysses, Bear, Craft).

- **Low Latency**: Keystroke-to-render latency must be imperceptible (<16ms target). Never laggy or sluggish.
- **Focus When You Get in the Zone**: Distraction-free writing mode, smooth focus transitions, no interruptions.
- **Excellent Typography and Design**: Beautiful, readable fonts with proper spacing, sizing, and contrast. Premium and delightful UI.
- **Extremely Flexible**: Mix components together—text, images, videos, embeds, code blocks, tables, AI chat threads—seamlessly within a single entry.

#### 2. Fast to Find Your Content

Search must be comprehensive, fast, and forgiving.

- **Searchable by Whatever You Can Remember**: Full-text search across all content, metadata search (tags, dates, attachments), media search.
- **Search is Fast So You Can Correct Quickly**: Sub-100ms search results. Instant feedback as you type.

#### 3. Privacy First

We are a private personal assistant and journaling app. Privacy is a core feature.

- **Your Private Personal Assistant**: Trusted, private space for thoughts. Local-first by default.
- **Stay at the Frontier of AI**: Leverage cutting-edge AI while maintaining privacy. Prioritize on-device AI models.
- **Transparency and Control**: Extremely transparent about data collection. User privacy comes first.
- **Privacy Does Not Mean Absolutely No Data Transfer**: Privacy means **predictable** and **transparent** data transfer with user control. Default to the most private option, requiring explicit opt-in for any data sharing.

### Product Scope

- **Audience**: Individuals who want a private, fast, local-first journal with optional AI assistance.
- **Platforms**: macOS desktop (Expo + React Native). Mobile later.
- **Connectivity**: Works fully offline. Optional backups to user-selected services.

#### Key Features

- **Entries + AI Convos**: Unified timeline combining manual journal entries and conversations with a personal AI.
- **Entry Types**: Journal Entry and AI Chat. _AI summarization planned._
- **Search**: Full-text search across all content. _Semantic (AI) search planned._
- **Encryption**: Local database encryption at rest (SQLCipher). _Zero-knowledge cloud backups planned._
- **Local AI**: On-device inference for AI conversations. _Summarization, Q&A, insight extraction planned._
- **Import/Export (Planned)**: JSON/Markdown export; portable backups.
- **Trial & Purchase (Planned)**: 7-day free trial, then $45 lifetime license.

#### Non-Goals (v1)

- Multi-user collaboration
- Realtime sync across devices (v1 supports backup/restore, not continuous sync)
- Server-side features; no vendor lock-in

#### Success Metrics

- **Performance**: <100ms search on 10k notes; <300ms app launch cold.
- **Reliability**: Zero data loss in local crash scenarios; validated backup/restore.
- **Delight**: >40% week-4 retention after trial; NPS > 50.

### UX Overview

#### Primary Screens

- **Home Timeline**: Mixed list of entries and AI convos, grouped by day. Quick filters: Entries, AI, Favorites, Tags.
- **Composer**:
  - Journal Entry: Rich block-based editor (checkboxes, lists, tables, images, embeds, code blocks) - WYSIWYG with low latency
  - AI Chat: Markdown-based editor/display
- **Search**: Unified search bar with filters. _Keyboard navigation and semantic tabs planned._
- **Settings**: Encryption, backups, AI model, license, import/export.

#### Interaction Principles

- **Zero friction**: Minimal modals; optimistic UI; autosave.
- **Accessible (Planned)**: _Full keyboard navigation, prefers-reduced-motion, and high contrast mode are planned._
- **Trust**: Clear encryption states; explicit backup confirmation; no hidden sync.

---

## Architecture

### Tech Stack

- **Framework**: Expo + React Native for macOS via [react-native-macos](https://github.com/microsoft/react-native-macos)
- **Package Manager**: pnpm (always use `pnpm` commands, never `npm` or `yarn`)
- **Build System**: Turborepo for monorepo task orchestration
- **No web app**: Desktop-only initially

### Monorepo Structure

```
journal/
├── apps/
│   ├── app/                  # Main Expo mobile app
│   │   ├── App.tsx
│   │   ├── index.ts
│   │   ├── lib/              # Application code
│   │   ├── modules/          # Native modules (platform-ai, widget-bridge, keyboard-module)
│   │   ├── packages/         # Swift package (widget-utils)
│   │   ├── assets/
│   │   ├── plugins/
│   │   ├── targets/          # @bacons/apple-targets
│   │   ├── native/
│   │   └── package.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deca-inc/jot](https://github.com/deca-inc/jot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
