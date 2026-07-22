---
trigger: always_on
description: > This document is mainly for agents and LLMs to follow when maintaining,
---

# React Native Skills

**Version 1.0.0**  
Engineering  
January 2026

> **Note:**  
> This document is mainly for agents and LLMs to follow when maintaining,  
> generating, or refactoring React Native codebases. Humans  
> may also find it useful, but guidance here is optimized for automation  
> and consistency by AI-assisted workflows.

---

## Abstract

Comprehensive performance optimization guide for React Native applications, designed for AI agents and LLMs. Contains 35+ rules across 13 categories, prioritized by impact from critical (core rendering, list performance) to incremental (fonts, imports). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and specific impact metrics to guide automated refactoring and code generation.

---

## Table of Contents

1. [Core Rendering](#1-core-rendering) — **CRITICAL**
   - 1.1 [Never Use && with Potentially Falsy Values](#11-never-use--with-potentially-falsy-values)
   - 1.2 [Wrap Strings in Text Components](#12-wrap-strings-in-text-components)
2. [List Performance](#2-list-performance) — **HIGH**
   - 2.1 [Avoid Inline Objects in renderItem](#21-avoid-inline-objects-in-renderitem)
   - 2.2 [Hoist callbacks to the root of lists](#22-hoist-callbacks-to-the-root-of-lists)
   - 2.3 [Keep List Items Lightweight](#23-keep-list-items-lightweight)
   - 2.4 [Optimize List Performance with Stable Object References](#24-optimize-list-performance-with-stable-object-references)
   - 2.5 [Pass Primitives to List Items for Memoization](#25-pass-primitives-to-list-items-for-memoization)
   - 2.6 [Use a List Virtualizer for Any List](#26-use-a-list-virtualizer-for-any-list)
   - 2.7 [Use Compressed Images in Lists](#27-use-compressed-images-in-lists)
   - 2.8 [Use Item Types for Heterogeneous Lists](#28-use-item-types-for-heterogeneous-lists)
3. [Animation](#3-animation) — **HIGH**
   - 3.1 [Animate Transform and Opacity Instead of Layout Properties](#31-animate-transform-and-opacity-instead-of-layout-properties)
   - 3.2 [Prefer useDerivedValue Over useAnimatedReaction](#32-prefer-usederivedvalue-over-useanimatedreaction)
   - 3.3 [Use GestureDetector for Animated Press States](#33-use-gesturedetector-for-animated-press-states)
4. [Scroll Performance](#4-scroll-performance) — **HIGH**
   - 4.1 [Never Track Scroll Position in useState](#41-never-track-scroll-position-in-usestate)
5. [Navigation](#5-navigation) — **HIGH**
   - 5.1 [Use Native Navigators for Navigation](#51-use-native-navigators-for-navigation)
6. [React State](#6-react-state) — **MEDIUM**
   - 6.1 [Minimize State Variables and Derive Values](#61-minimize-state-variables-and-derive-values)
   - 6.2 [Use fallback state instead of initialState](#62-use-fallback-state-instead-of-initialstate)
   - 6.3 [useState Dispatch updaters for State That Depends on Current Value](#63-usestate-dispatch-updaters-for-state-that-depends-on-current-value)
7. [State Architecture](#7-state-architecture) — **MEDIUM**
   - 7.1 [State Must Represent Ground Truth](#71-state-must-represent-ground-truth)
8. [React Compiler](#8-react-compiler) — **MEDIUM**
   - 8.1 [Destructure Functions Early in Render (React Compiler)](#81-destructure-functions-early-in-render-react-compiler)
   - 8.2 [Use .get() and .set() for Reanimated Shared Values (not .value)](#82-use-get-and-set-for-reanimated-shared-values-not-value)
9. [User Interface](#9-user-interface) — **MEDIUM**
   - 9.1 [Measuring View Dimensions](#91-measuring-view-dimensions)
   - 9.2 [Modern React Native Styling Patterns](#92-modern-react-native-styling-patterns)
   - 9.3 [Use contentInset for Dynamic ScrollView Spacing](#93-use-contentinset-for-dynamic-scrollview-spacing)
   - 9.4 [Use contentInsetAdjustmentBehavior for Safe Areas](#94-use-contentinsetadjustmentbehavior-for-safe-areas)
   - 9.5 [Use expo-image for Optimized Images](#95-use-expo-image-for-optimized-images)
   - 9.6 [Use Galeria for Image Galleries and Lightbox](#96-use-galeria-for-image-galleries-and-lightbox)
   - 9.7 [Use Native Menus for Dropdowns and Context Menus](#97-use-native-menus-for-dropdowns-and-context-menus)
   - 9.8 [Use Native Modals Over JS-Based Bottom Sheets](#98-use-native-modals-over-js-based-bottom-sheets)
   - 9.9 [Use Pressable Instead of Touchable Components](#99-use-pressable-instead-of-touchable-components)
10. [Design System](#10-design-system) — **MEDIUM**

- 10.1 [Use Compound Components Over Polymorphic Children](#101-use-compound-components-over-polymorphic-children)

11. [Monorepo](#11-monorepo) — **LOW**

- 11.1 [Install Native Dependencies in App Directory](#111-install-native-dependencies-in-app-directory)
- 11.2 [Use Single Dependency Versions Across Monorepo](#112-use-single-dependency-versions-across-monorepo)

12. [Third-Party Dependencies](#12-third-party-dependencies) — **LOW**

- 12.1 [Import from Design System Folder](#121-import-from-design-system-folder)

13. [JavaScript](#13-javascript) — **LOW**

- 13.1 [Hoist Intl Formatter Creation](#131-hoist-intl-formatter-creation)

14. [Fonts](#14-fonts) — **LOW**

- 14.1 [Load fonts natively at build time](#141-load-fonts-natively-at-build-time)

---

## 1. Core Rendering

**Impact: CRITICAL**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gronxb/codex-relay](https://github.com/gronxb/codex-relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
