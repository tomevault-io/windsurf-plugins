---
trigger: always_on
description: Use when sequential user actions hit multiple endpoints needing the same data within seconds.
---

# React Best Practices

**Version 1.0.0**  
Vercel Engineering  
January 2026

> **Note:**  
> This document is mainly for agents and LLMs to follow when maintaining,  
> generating, or refactoring React and Next.js codebases. Humans  
> may also find it useful, but guidance here is optimized for automation  
> and consistency by AI-assisted workflows.

---

## Abstract

Comprehensive performance optimization guide for React and Next.js applications, designed for AI agents and LLMs. Contains 40+ rules across 8 categories, prioritized by impact from critical (eliminating waterfalls, reducing bundle size) to incremental (advanced patterns). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and specific impact metrics to guide automated refactoring and code generation.

---

## Table of Contents

1. [Eliminating Waterfalls](#1-eliminating-waterfalls) — **CRITICAL**
   - 1.1 [Defer Await Until Needed](#11-defer-await-until-needed)
   - 1.2 [Dependency-Based Parallelization](#12-dependency-based-parallelization)
   - 1.3 [Prevent Waterfall Chains in API Routes](#13-prevent-waterfall-chains-in-api-routes)
   - 1.4 [Promise.all() for Independent Operations](#14-promiseall-for-independent-operations)
   - 1.5 [Strategic Suspense Boundaries](#15-strategic-suspense-boundaries)
2. [Bundle Size Optimization](#2-bundle-size-optimization) — **CRITICAL**
   - 2.1 [Avoid Barrel File Imports](#21-avoid-barrel-file-imports)
   - 2.2 [Conditional Module Loading](#22-conditional-module-loading)
   - 2.3 [Defer Non-Critical Third-Party Libraries](#23-defer-non-critical-third-party-libraries)
   - 2.4 [Dynamic Imports for Heavy Components](#24-dynamic-imports-for-heavy-components)
   - 2.5 [Preload Based on User Intent](#25-preload-based-on-user-intent)
3. [Server-Side Performance](#3-server-side-performance) — **HIGH**
   - 3.1 [Authenticate Server Actions Like API Routes](#31-authenticate-server-actions-like-api-routes)
   - 3.2 [Avoid Duplicate Serialization in RSC Props](#32-avoid-duplicate-serialization-in-rsc-props)
   - 3.3 [Cross-Request LRU Caching](#33-cross-request-lru-caching)
   - 3.4 [Hoist Static I/O to Module Level](#34-hoist-static-io-to-module-level)
   - 3.5 [Minimize Serialization at RSC Boundaries](#35-minimize-serialization-at-rsc-boundaries)
   - 3.6 [Parallel Data Fetching with Component Composition](#36-parallel-data-fetching-with-component-composition)
   - 3.7 [Per-Request Deduplication with React.cache()](#37-per-request-deduplication-with-reactcache)
   - 3.8 [Use after() for Non-Blocking Operations](#38-use-after-for-non-blocking-operations)
4. [Client-Side Data Fetching](#4-client-side-data-fetching) — **MEDIUM-HIGH**
   - 4.1 [Deduplicate Global Event Listeners](#41-deduplicate-global-event-listeners)
   - 4.2 [Use Passive Event Listeners for Scrolling Performance](#42-use-passive-event-listeners-for-scrolling-performance)
   - 4.3 [Use SWR for Automatic Deduplication](#43-use-swr-for-automatic-deduplication)
   - 4.4 [Version and Minimize localStorage Data](#44-version-and-minimize-localstorage-data)
5. [Re-render Optimization](#5-re-render-optimization) — **MEDIUM**
   - 5.1 [Calculate Derived State During Rendering](#51-calculate-derived-state-during-rendering)
   - 5.2 [Defer State Reads to Usage Point](#52-defer-state-reads-to-usage-point)
   - 5.3 [Do not wrap a simple expression with a primitive result type in useMemo](#53-do-not-wrap-a-simple-expression-with-a-primitive-result-type-in-usememo)
   - 5.4 [Extract Default Non-primitive Parameter Value from Memoized Component to Constant](#54-extract-default-non-primitive-parameter-value-from-memoized-component-to-constant)
   - 5.5 [Extract to Memoized Components](#55-extract-to-memoized-components)
   - 5.6 [Narrow Effect Dependencies](#56-narrow-effect-dependencies)
   - 5.7 [Put Interaction Logic in Event Handlers](#57-put-interaction-logic-in-event-handlers)
   - 5.8 [Subscribe to Derived State](#58-subscribe-to-derived-state)
   - 5.9 [Use Functional setState Updates](#59-use-functional-setstate-updates)
   - 5.10 [Use Lazy State Initialization](#510-use-lazy-state-initialization)
   - 5.11 [Use Transitions for Non-Urgent Updates](#511-use-transitions-for-non-urgent-updates)
   - 5.12 [Use useRef for Transient Values](#512-use-useref-for-transient-values)
6. [Rendering Performance](#6-rendering-performance) — **MEDIUM**
   - 6.1 [Animate SVG Wrapper Instead of SVG Element](#61-animate-svg-wrapper-instead-of-svg-element)
   - 6.2 [CSS content-visibility for Long Lists](#62-css-content-visibility-for-long-lists)
   - 6.3 [Hoist Static JSX Elements](#63-hoist-static-jsx-elements)
   - 6.4 [Optimize SVG Precision](#64-optimize-svg-precision)
   - 6.5 [Prevent Hydration Mismatch Without Flickering](#65-prevent-hydration-mismatch-without-flickering)
   - 6.6 [Suppress Expected Hydration Mismatches](#66-suppress-expected-hydration-mismatches)
   - 6.7 [Use Activity Component for Show/Hide](#67-use-activity-component-for-showhide)
   - 6.8 [Use Explicit Conditional Rendering](#68-use-explicit-conditional-rendering)
   - 6.9 [Use useTransition Over Manual Loading States](#69-use-usetransition-over-manual-loading-states)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mmmtttt/ULTIMATE_WEB](https://github.com/Mmmtttt/ULTIMATE_WEB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
