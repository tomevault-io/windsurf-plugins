---
trigger: always_on
description: Write readable, intention-revealing code with precise names, flat control flow, consistent abstraction levels, focused files, named constants, formatter-aligned style, repository conventions, and testable seams. Use when naming feels off, logic is hard to follow, functions do too much, nested conditionals obscure flow, files mix many exports, magic numbers are inline, formatting is inconsistent, logic is reimplemented at call sites, tests overuse mocks or dependency injection, or a refactor shou
---


# Code Clarity Framework

A practical framework for writing code that communicates intent clearly. The central thesis: **a developer reads code far more than they write it**, so every naming and structural decision is a communication decision. Code that requires a reader to reconstruct the author's mental model has failed at its primary job.

This framework focuses on the micro-level of software design — the decisions made at the function, method, class, and file level — and complements macro-level architecture thinking.

**Languages covered.** The framework is language-agnostic in principle, with first-class guidance for **Swift**, **TypeScript**, and **Electron** (the main/preload/renderer split), plus Go and Python equivalents where they sharpen a point. Swift examples carry the value-vs-identity and `guard` material; TypeScript and Electron examples carry the one-object-per-file, module-constant, discriminated-union, typed-IPC, and formatter-consistency material drawn from real production codebases.

## Core Principles

**Code is written once but read hundreds of times.** Every name, every function boundary, every conditional structure is a message to the next reader (usually yourself, six months later). Clarity is not a style preference — it is a correctness property: unclear code is one misunderstanding away from a bug.

**Clarity is partly local to a repository.** A refactor that ignores a codebase's existing naming, file organization, and control-flow habits can make the result less readable even when the individual function improves in isolation.

## Scoring

**Goal: 10/10.** When reviewing or writing code, rate it 0–10 on clarity. A 10/10 has names that read like prose, functions that do exactly one thing at one level of abstraction, guard clauses that eliminate nesting, structures whose responsibilities are obvious from their name alone, and changes that match the repository's established local conventions. Provide the current score and exactly what to change to reach 10/10.

## The Code Clarity Framework

Thirteen principles for writing code that communicates clearly. Principles 1–8 are structural and language-agnostic; principles 9–12 cover file organization, named constants, mechanical formatting consistency, and the Electron process boundary — the areas most often neglected in TypeScript and Electron codebases; principle 13 covers dependencies and seam design — one canonical implementation, seams only where behavior truly varies, and testing without drowning in mocks and dependency injection:

---

### 1. Naming: Names Are Your Primary API

**Core concept:** Every name — variable, function, method, class, parameter — is a contract with the reader. Names should reveal intent, not implementation. The reader should never need to read a function's body to understand what it does or what a variable contains.

**Why it works:** In most codebases, 70%+ of identifiers are custom names. If those names are precise, the code reads like a domain description. If they are vague or misleading, every reader carries extra cognitive load reconstructing what the author meant.

**Key insights:**
- Functions that *do something* use verbs: `fetchUser()`, `validateInput()`, `syncViews()`
- Functions that *return a value* describe what they return: `activeUsers()`, `formattedDate()`, `bytesReceived()`
- Booleans use `is/has/can/should`: `isVisible`, `hasChildren`, `canSubmit`, `shouldRetry`
- Avoid double negatives: `hasElements` not `!isEmpty`, `isEnabled` not `!isDisabled`
- Class methods drop redundant type context: `line.length()` not `line.getLineLength()`
- Names should be proportional to scope: loop variables can be `i`, module-level state needs full names
- Abbreviations are only valid if universally understood in the domain (`url`, `id`, `dto`)

**Code applications:**

| Context | Unclear | Clear |
|---------|---------|-------|
| **Action function** | `handle()`, `process()`, `manage()` | `submitOrder()`, `parseResponse()`, `invalidateCache()` |
| **Return-value function** | `get()`, `fetch()`, `data()` | `currentUser()`, `pendingRequests()`, `errorMessage()` |
| **Boolean** | `flag`, `check`, `status`, `valid` | `isAuthenticated`, `hasUnreadMessages`, `canRetry` |
| **Swift bool** | `!list.isEmpty` | `list.hasElements` (extension) |
| **TS bool** | `if (!user.disabled)` | `if (user.isEnabled)` |
| **Class name** | `Manager`, `Handler`, `Helper`, `Util` | `RequestThrottler`, `TokenRefresher`, `PayloadEncoder` |
| **TS type suffix** | `data`, `info`, `thing` | role suffixes: `*Service`, `*Store`, `*Schema`, `*Registry`, `*Queue` |
| **Parameter** | `func send(_ data: Data, _ b: Bool)` | `func send(_ payload: Data, encrypted: Bool)` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lakr233/code-clarity](https://github.com/Lakr233/code-clarity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
