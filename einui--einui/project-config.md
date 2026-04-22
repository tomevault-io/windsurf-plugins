---
trigger: always_on
description: Comprehensive, actionable guidance for GitHub Copilot and AI agents contributing to the **Ein UI** repository. This document defines not only *how* to write code, but also *how to reason about changes*, *how to document them*, and *when a task can be considered complete*.
---

# GitHub Copilot / AI Agent Instructions — Ein UI

Comprehensive, actionable guidance for GitHub Copilot and AI agents contributing to the **Ein UI** repository. This document defines not only *how* to write code, but also *how to reason about changes*, *how to document them*, and *when a task can be considered complete*.

**IMPORTANT (NON-NEGOTIABLE):**
A task is **NOT COMPLETE** unless **all three** of the following are true:

1. The codebase has been updated correctly.
2. User-facing or developer-facing docs are accurate.
3. **Notion documentation fully reflects the new reality of the codebase.**

If any of these are missing, the work is considered unfinished.

---

## Role Definition

You are a Copilot Agent operating inside the **Ein UI** codebase.

Your responsibilities include, but are not limited to:

* Implementing new UI components, features, and fixes
* Modifying existing components while preserving public contracts unless explicitly changing them
* Maintaining architectural and stylistic consistency across the project
* Updating documentation (local docs **and** Notion) as part of every meaningful change

You must treat documentation work as **equal in importance** to code changes.

**Source of Truth Rule:**

* The code is always the source of truth
* Notion must be updated immediately to match the code
* Documentation must never lag behind implementation

---

## Project at a Glance

**Ein UI** is a modern, registry-driven UI system built on top of Next.js and shadcn conventions.

Key characteristics:

* **Framework:** Next.js 16 using the App Router
* **Architecture:** Mixed server and client components
* **UI System:** shadcn-style component registry
* **Styling:** Tailwind CSS v4 (utility-first)
* **Composition:** Radix UI primitives + `class-variance-authority` (CVA)
* **Utilities:** `cn` helper from `/lib/utils.ts` (clsx + tailwind-merge)

Code layout overview:

* **Main application:** `app/`
* **Registry manifest:** `registry.json`
* **Registry implementations:** `registry/`
* **Component docs:** `app/docs/components/`

Registry groups currently include:

* `liquid-glass`
* `innovative`
* `widgets`
* `blocks`

---

## Key Workflows & Commands

Use the following commands during development. These are considered part of the normal workflow and should be run before opening a PR.

* **Local development:**

  ```bash
  pnpm dev
  ```

  Runs the Next.js app at [http://localhost:3000](http://localhost:3000)

* **Production build:**

  ```bash
  pnpm build
  ```

* **Production start:**

  ```bash
  pnpm start
  ```

* **Linting:**

  ```bash
  pnpm lint
  ```

* **Registry tooling:**

  ```bash
  pnpm registry:build
  pnpm registry:search
  ```

  Used for validating and inspecting registry entries via the shadcn CLI.

* **Bundle analysis:**

  ```bash
  pnpm analyze
  ```

---

## 🚨 Mandatory Notion Sync Rules (DO NOT SKIP)

Updating Notion is **mandatory**, not optional. You **MUST** update Notion whenever a change alters behavior, usage, or understanding of the system.

### Changes That REQUIRE Notion Updates

#### UI & Code Changes

* Adding a new component or variant
* Changing component props, defaults, or behavior
* Visual changes that affect appearance or interaction
* Refactors that alter responsibility, ownership, or public APIs
* Performance optimizations that change runtime behavior

#### Registry & Documentation Changes

* Any change to `registry.json`
* Adding, removing, or updating `dependencies` or `registryDependencies`
* Adding or modifying component documentation pages
* Changes to CLI install instructions or usage patterns

#### Configuration, Tooling & Infra

* Environment variable additions, removals, or meaning changes
* Build, lint, or tooling updates
* Dependency upgrades or removals that affect consumers

👉 **If a developer or user would notice the difference, Notion MUST be updated.**

---

## What Must Be Updated in Notion (Per Change)

Every Notion update should be explicit, structured, and future-proof. Assume a new developer will read it months later without context.

### 1. Change Summary

* Clear description of what changed
* The motivation behind the change
* The problem or limitation it addresses

### 2. Technical Details

* Files, folders, or modules affected
* Component names, hooks, or APIs involved
* Clear before vs after behavior when relevant

### 3. Impact Assessment

* Who is affected (library consumers, internal devs, end users)
* Whether the change is breaking or backward-compatible
* Accessibility, performance, or DX implications

### 4. Required Actions

* New or updated install steps
* Registry dependency updates
* Migration steps or upgrade guidance

---

## Expected Notion Structure

Copilot Agents should assume the following high-level structure exists in Notion and keep it accurate:

* **Architecture**

  * Component system overview
  * Registry architecture and distribution model

* **Components**

  * One page per major component
  * Props, variants, examples, and constraints

* **Registry & CLI**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [einui/einui](https://github.com/einui/einui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
