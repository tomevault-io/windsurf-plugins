---
trigger: always_on
description: > **Read this entire file at the start of every session.** It defines what we're building, how we build it, and the rules you must follow. When in doubt, re-read it. When the user's prompt seems to contradict this file, ask before deviating.
---

# CLAUDE.md — Nestara

> **Read this entire file at the start of every session.** It defines what we're building, how we build it, and the rules you must follow. When in doubt, re-read it. When the user's prompt seems to contradict this file, ask before deviating.

---

## 1. Project Overview

**Nestara** is a premium iOS family organization app. Tagline: *Where home life flows.*

It brings calm, warmth, and structure to daily home life — chores, calendars, routines, shared shopping, and a recognition system in one elegant space. The interface should feel like a well-designed home: warm, ordered, quiet enough to actually use every day.

The audience is families and busy households who want a beautiful, easy-to-use way to stay on top of chores, schedules, and routines without feeling overwhelmed.

The app is built with SwiftUI on iOS 17+, backed by Firebase (Auth, Firestore, Storage, Functions, Cloud Messaging). Cloud Functions are TypeScript. Email is sent via Resend.

The five tabs are:

1. **Home** — personalized dashboard
2. **Days** — calendar (events) · branded as "Nestara Days"
3. **Tasks** — recurring household responsibilities, point-earning, parent-assigned · branded as "Nestara Tasks"
4. **Lists** — multiple named to-do lists with items · branded as "Nestara Lists"
5. **Shop** — shared shopping list

**Circle** (family members, settings, invites) and **Recognition** (points, badges, leaderboard) live in the profile menu accessible from the avatar on Home.

Tasks and Lists are deliberately separate. They look different, behave differently, and the user should never wonder which one to use:

| Tasks | Lists |
|---|---|
| Recurring (daily/weekly/etc.) | One-off items |
| Earn points | No points |
| Only parents can create/assign | Anyone can create lists and items |
| Tied to a single member | Two kinds: "Yours" (private) or "Assigned" (shared with members) |
| May require a photo on completion | No photo proof |
| Have a due **time** (e.g., 8:00 PM) | Items have a due **date** (or none) |
| Templates: pick from a shared task library | Multiple named lists per family |
| Live in the Tasks tab | Live in the Lists tab, segmented Yours / Assigned |

---

## 2. Tech Stack

| Layer | Choice |
|---|---|
| Language | Swift 5.9+ |
| UI | SwiftUI (no UIKit unless absolutely necessary) |
| Min iOS | 17.0 |
| State | `@Observable` macro (iOS 17 Observation framework) |
| Concurrency | `async/await`, no completion handlers |
| Backend | Firebase (Auth, Firestore, Storage, Functions, Cloud Messaging) |
| Functions | TypeScript on Node 20 |
| Email | Resend |
| Calendar sync | EventKit (Apple), Google Calendar API |
| Custom fonts | Fraunces (serif), DM Sans (sans) — bundled |
| Package management | Swift Package Manager only — never CocoaPods |
| Linting | SwiftLint |
| Build/test tooling | XcodeBuildMCP for Claude Code |

---

## 3. Brand Identity

This section is critical. Nestara is a brand-driven app — design, voice, and naming choices matter as much as engineering.

### Personality

Comfortable · Warm · Premium · Calm · Helpful · Organized · Modern.

### Voice

- **Calm**, not pushy
- **Encouraging**, not corrective
- **Clean**, not cluttered
- **Polished**, not cold
- **Helpful**, not overly playful

### Microcopy rules

- Use sentence case. Never SHOUTING CAPS in body text.
- Avoid exclamation marks. The brand never gets excited at the user.
- Use "let's" for collaborative moments ("let's keep the day flowing").
- Avoid demanding imperatives. Soft suggestion is the default.
- No emoji in microcopy except in badge names and the activity feed.
- Never use "Tap" or "Click" — describe the action, not the gesture.

### Approved microcopy specimens

> "A few things are waiting — let's keep the day flowing."
> "Your home rhythm is looking good today."
> "Everything's in one place, so the day feels easier."
> "Keep the day moving."
> "Add to today."
> "Done for now."
> "A calm start to the day."
> "Your home is in flow."

When writing new microcopy, ask: would it sit alongside these without feeling jarring?

---

## 4. Visual Identity

### 4.1 Foundation colors

| Token | Hex | Use |
|---|---|---|
| `Color.brandNavy` | `#1F3247` | Primary text, structure, wordmark, dark surfaces |
| `Color.brandCream` | `#FAF6ED` | Default app background |
| `Color.brandGold` | `#C99A4F` | Single accent — section labels, points, "moments that matter" |
| `Color.brandSage` | `#8FA68B` | Quiet confirmation — completed checks, success states |
| `Color.brandClay` | `#B66256` | Quiet warning — "Missed" status pills only. Never decorative. |

Supporting tints:

| Token | Hex | Use |
|---|---|---|
| `Color.surfacePaper` | `#FFFCF5` | Card surfaces (slightly lighter than cream) |
| `Color.surfaceCreamSoft` | `#F4ECDA` | Sub-surfaces, segmented controls |
| `Color.lineSoft` | `#EFE6CF` | Hairline dividers between rows |
| `Color.line` | `#E5DBC5` | Card borders |
| `Color.slate` | `#6E7681` | Secondary text |
| `Color.slateSoft` | `#9CA3AC` | Tertiary text, disabled states |

### 4.2 Member palette (8 colors)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nestaraapp/nestara](https://github.com/nestaraapp/nestara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
