---
trigger: always_on
description: > **Internal engineering documentation for a premium offline-first student productivity workspace.**
---

# Student OS — Complete Technical & Product Overview

> **Internal engineering documentation for a premium offline-first student productivity workspace.**
>
> Built by Jainam Karnawat. Current version: 1.0.0. Stack: Next.js 16 + React 19 + TypeScript + Tailwind CSS v4 + Framer Motion 12 + Zustand 5.

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [User Experience Philosophy](#2-user-experience-philosophy)
3. [Architecture Overview](#3-architecture-overview)
4. [Codebase Structure](#4-codebase-structure)
5. [Feature Systems](#5-feature-systems)
6. [State Management & Persistence](#6-state-management--persistence)
7. [Design System](#7-design-system)
8. [Motion & Animation System](#8-motion--animation-system)
9. [Performance Strategy](#9-performance-strategy)
10. [Mobile Strategy & PWA](#10-mobile-strategy--pwa)
11. [App Lifecycle & Routing](#11-app-lifecycle--routing)
12. [Known Limitations & Future Roadmap](#12-known-limitations--future-roadmap)
13. [New Developer Onboarding Guide](#13-new-developer-onboarding-guide)

---

## 1. Product Overview

### What It Is

Student OS is a **premium, offline-first, single-page productivity workspace** designed specifically for students. It replaces the need for multiple disjointed tools (Pomodoro timer, task manager, habit tracker, calendar, note-taker, analytics dashboard) with a single cohesive, calm, and focused experience.

### The Vision

Most student productivity tools fall into two camps:
- **Over-engineered**: Notion-style infinite flexibility with crippling setup friction
- **Under-designed**: Bare-bones todo lists with no motivation system, no study-specific features, and no emotional consideration

Student OS occupies the middle ground: **opinionated but not rigid, premium but not complex, minimal but not sparse**.

The core belief is that **consistency beats intensity**. The app is designed to be used daily for months and years, not as a novelty that wears off after a week.

### Target Users

- University/college students managing multiple subjects
- Self-taught learners structuring independent study
- Students preparing for competitive exams
- Anyone who needs deep work sessions, habit tracking, and academic organization

### Core Values

| Value | How it Manifests |
|-------|------------------|
| **Calm productivity** | No notifications, no noise, no real-time collaboration pressure |
| **Local-first** | Your data stays on your device. No account, no server, no privacy concerns |
| **Premium feel** | OKLCH color space, glass morphism, subtle animations, thoughtful typography |
| **Deep focus** | Focus mode is the emotional centerpiece — a premium timer + ambient environment |
| **Long-term consistency** | Streaks, XP, habits, analytics — all designed to reward showing up daily |
| **Lightweight** | ~100KB of JS per view (lazy-loaded), no heavy 3D, no external API calls |

### What It Is NOT

- Not a collaboration tool. No real-time sync, no multiplayer, no sharing.
- Not a note-taking app (no rich text editor, no markdown support... yet).
- Not cloud-dependent. Everything works offline from install.
- Not a gamification trap. XP is earned, not purchased. There is no way to "cheat the system."

---

## 2. User Experience Philosophy

### Emotional Design Goals

Every screen in Student OS is designed around a specific emotional state:

| Screen | Emotional Goal | Design Strategy |
|--------|----------------|-----------------|
| **Dashboard** | Grounded, motivated | Greeting by time-of-day, daily quote, visible progress, heatmap of consistency |
| **Focus** | Calm, centered, undistractable | Dark ambient gradient, no UI chrome, timer as singular focal point, no notifications |
| **Tasks** | Clear, organized, in control | Clean list/kanban, priority badges, due dates, subtasks, one-click complete |
| **Habits** | Encouraged, consistent | Visual streak grid, celebratory animations on toggle, stat cards for wins |
| **Planner** | Prepared, intentional | Calendar with study/event types, hourly timeline view |
| **Analytics** | Reflective, insightful | Charts showing trends, not judgments; AI-like insights (e.g., "You focus best in the morning") |
| **Settings** | Safe, in control of data | Export/import, clear data, theme customization, avatar personalization |

### UX Principles

1. **Zero forced interruption.** No push notifications, no "reminders," no badges. The user comes to the app on their terms.
2. **Every interaction is a choice.** The app suggests (presets, quotes, priority levels) but never mandates.
3. **Motion must earn its place.** Animations are either functional (page transitions, state feedback) or ambient (subtle floating orbs, breathing timer ring). Never decorative noise.
4. **Data is sacred.** Local-first means local-only. Export is one click. There is always a path out.
5. **The first run sets the tone.** The onboarding screen is a glass panel with animated orbs, a single text input for the name, and the line "Your name stays on this device. No account needed."

---

## 3. Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                      Root Layout                     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jainam662010-boop/OS](https://github.com/jainam662010-boop/OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
