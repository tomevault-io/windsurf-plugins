---
trigger: always_on
description: This file provides guidance to Claude Code when working with this Nostr UX pattern library repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this Nostr UX pattern library repository.

---

## ⚠️ CRITICAL RULES - READ FIRST ⚠️

### HEXTRA THEME MODIFICATIONS - ABSOLUTELY FORBIDDEN

**YOU ARE STRICTLY FORBIDDEN FROM MAKING ANY CUSTOM MODIFICATIONS TO THE HEXTRA THEME WITHOUT EXPRESS PERMISSION.**

**MANDATORY WORKFLOW:**

1. **ALWAYS CHECK THE OFFICIAL HEXTRA DOCUMENTATION FIRST** at https://imfing.github.io/hextra/docs/
2. **ALWAYS USE BUILT-IN HEXTRA SHORTCODES** - Never create custom HTML, CSS, or layout overrides
3. **NEVER CREATE FILES IN `layouts/` DIRECTORY** - This overrides the theme and is forbidden
4. **NEVER ADD CUSTOM CSS** unless it's for the Hextra-documented purple theme configuration
5. **NEVER MODIFY THEME BEHAVIOR** - Use what the theme provides or ask for permission first

**Available Hextra Shortcodes (USE THESE):**
- `{{< cards >}}` and `{{< card >}}` - For card grids
- `{{< callout >}}` - For notices
- `{{< details >}}` - For collapsible content
- `{{< steps >}}` - For step-by-step guides
- `{{< tabs >}}` - For tabbed content

**If you need to do something and don't know the Hextra way:**
1. Search the Hextra documentation at https://imfing.github.io/hextra/docs/
2. Use WebFetch to read the relevant docs page
3. Use what the theme provides
4. If theme doesn't support it, ASK THE USER before doing anything custom

**Violation of these rules will result in immediate termination.**

---

## Project Overview

**Repository:** nostr-ux-patterns
**Purpose:** UX pattern library providing evidence-informed best practices for Nostr consumer applications
**Owner:** Shawn Yeager (@shawnyeager)
**Target Audience:** Nostr developers building consumer apps (social clients)

**Core Thesis:** Great UX is the gateway to the protocol's power. Ship working experiences, then add features.

---

## Project Goals & Context

### Primary Goal

Help existing Nostr developers improve their UX, while being useful enough for mainstream developers to understand Nostr app development.

### Target Frustrations Being Addressed

1. **Naive feature bloat:** Piling on features without user validation
2. **Protocol purism:** Thinking that protocol purity will somehow win when the UX is poor

### Success Criteria

We'll know this is valuable based on **the discussion it generates on Nostr** itself.

---

## Scope & Approach (From Initial Interview)

### Depth vs Breadth

- **Deep dive on 5-7 critical patterns** (not broad coverage of 15-20)
- Patterns chosen based on research of Nostr apps' typical weaknesses
- Each pattern: thorough treatment with examples, anti-patterns, validation

### Content Balance

- **70%:** Universal social app UX principles (applicable to any platform)
- **30%:** Nostr-specific considerations (relays, keys, zaps, censorship-resistance)

### Audience Profile

- **Mix:** Mobile developers (iOS/Android) + Web developers (React, Vue, etc.)
- **Level:** Mid to senior developers (junior devs have bigger problems)
- **Nostr Knowledge:** Assume they know basics of Nostr protocols

### Tone & Style

- **Balanced:** Principles + rationale (not purely academic, not purely prescriptive)
- Evidence-based with citations where useful
- Opinionated where it helps ("just do this" when appropriate)
- Practical over theoretical

### Length Target

"Just make it good" - no specific page count constraints

---

## Research Findings (Foundation for the 6 Patterns)

### Primary Issues Identified

**Retention Crisis:**

- 30-day retention trends to 0% for recent cohorts
- Daily active users stuck at 10,000-12,000 "trusted" pubkeys
- Users need 5-6 different clients to work around bugs
- All apps described as "alpha state"

**Onboarding Failure:**

- 15-20 minute setup process
- Key management overwhelming and terrifying (loss = permanent identity loss)
- Relay selection too complex for new users
- Users abandon before reaching value

**Content Discovery Problems:**

- "Traditional apps win by having much better content selection"
- Cold start problem: empty feeds after signup
- No effective discovery mechanisms
- Chicken/egg: need users for content, need content for users

**Core Reliability Issues:**

- Posts disappearing seconds after posting
- Cross-client data loss (following/followers disappear when switching apps)
- Missing notifications
- Apps hanging/buffering
- Crashes and performance problems

**Complexity Exposure:**

- Protocol complexity exposed to end users
- Relay management shown to beginners
- Signer setup (NIP-46) too complex for onboarding
- Tech jargon in user-facing UI

**No Growth Loops:**

- No email invites
- Users not notified when tagged
- No habit formation triggers

### Root Causes

- Feature bloat without validating core UX works
- Protocol complexity exposed to end users
- Shipping features before core interactions are reliable
- Multi-relay coordination without optimization
- Power user features treated as essential
- Poor relay coordination and data validation

---

## The 6 Critical Patterns

Based on research findings, these patterns address the highest-impact UX problems:

1. **Onboarding & First-Run Experience**
   - Problem: 15-20 min setup, key management overwhelming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shawnyeager/nostr-ux-patterns](https://github.com/shawnyeager/nostr-ux-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
