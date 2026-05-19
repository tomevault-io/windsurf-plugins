---
trigger: always_on
description: This file is the working guide for any coding or documentation agent operating in this repository.
---

# AGENTS.md

This file is the working guide for any coding or documentation agent operating in this repository.

## Repository Purpose

This is a DSA practice repository made up almost entirely of Markdown notes.

The repo is not a runnable application. It is a study system for:

- documenting algorithm patterns
- storing problem-specific solutions
- comparing brute-force, better, and optimal approaches
- keeping implementation examples, mostly in Java
- preserving reasoning, intuition, and complexity analysis for interview prep

The current README describes the repo as a curated DSA practice collection focused on clear explanations, complexity analysis, and clean solutions.

## What Actually Exists Here

At the time of writing, the repo contains `81` Markdown files and no normal source-code package structure such as `src/`, `tests/`, `package.json`, `pom.xml`, or `requirements.txt`.

Key top-level items:

- `README.md`: short repository description
- `CLAUDE.md`: existing AI guidance, but incomplete and slightly generic
- `AGENTS.md`: this file
- topic notes at the root:
  - `two-pointers.md`
  - `sliding-window.md`
  - `prefix-sum.md`
  - `kadanes-algorithm.md`
- problem-set folders:
  - `Algorithms-Patterns/`
  - `Leetcode-150/`
  - `Striver/`
  - `RisingBrain/`
  - `padho-with-pratyush/`

There are also editor/project folders like `.idea/` and `.vscode/`, but they are not part of the learning content.

## Content Map

### 1. Root-Level Pattern Notes

The root contains broad algorithm-pattern explainers such as:

- `two-pointers.md`
- `sliding-window.md`
- `prefix-sum.md`
- `kadanes-algorithm.md`

These are generally more structured than the problem notes. They often include:

- a definition
- when to use the pattern
- why it works
- common variations
- a simple example
- differences from related techniques
- complexity summary

When editing these files, preserve their more educational and concept-driven style.

### 2. `Algorithms-Patterns/`

This folder is a second pattern library and includes files like:

- `Algorithms-Patterns/graph.md`
- `Algorithms-Patterns/heap.md`
- `Algorithms-Patterns/two-pointers.md`
- `Algorithms-Patterns/sliding-window.md`
- `Algorithms-Patterns/prefix-sum.md`
- `Algorithms-Patterns/kadanes-algorithm.md`
- `Algorithms-Patterns/two-sum.md`
- `Algorithms-Patterns/Tree/BinaryTree.md`
- `Algorithms-Patterns/Tree/levelOrder.md`

This area leans toward conceptual explanations and pattern-first thinking rather than only one problem at a time.

Use this folder when the content is about:

- a reusable technique
- a core data structure pattern such as heap / priority queue
- a general traversal pattern
- a problem family
- a template or recognition framework

Do not add single LeetCode problem notes here unless the note is explicitly teaching a pattern through that example.

### 3. `Leetcode-150/`

This folder contains compact notes for Top Interview 150 style problems, such as:

- `RemoveElement.md`
- `MergeSortedArray.md`
- `majority-element.md`
- `rotate-array.md`
- `best-time-to-buy-and-sell-stock.md`

These notes are typically short and direct:

- problem link
- one or more named solutions
- numbered steps
- Java code
- time and space complexity

This folder is suitable for concise problem-by-problem documentation.

### 4. `Striver/`

This folder currently focuses on Striver-style array content, for example:

- `Striver/Array/rotate-array.md`
- `Striver/Array/spiral-matrix.md`
- `Striver/Array/set-matrix-zeros.md`
- `Striver/Array/majority-element-ii.md`
- `Striver/Array/longestConsecutiveSeq.md`

Use this area for notes aligned with Striver’s sequencing or naming conventions. Preserve existing naming even when filenames are inconsistent with strict slug formatting.

### 5. `RisingBrain/`

This currently includes array/two-pointer material such as:

- `RisingBrain/Array/Two-Pointers/3sum.md`
- `RisingBrain/Array/Two-Pointers/container-with-most-water.md`
- `RisingBrain/Array/Two-Pointers/trapping-rain-water.md`
- `RisingBrain/Array/Two-Pointers/two-sum-ii-input-array-is-sorted.md`

Use this folder when content is tied to that learning track or grouping.

### 6. `padho-with-pratyush/`

This is one of the largest content sections and includes:

- `Binary-Search/`
- `Heap/`
- `Prefix-Sum/`
- `Recursion/`
- `Tree/`

Representative files:

- `padho-with-pratyush/Binary-Search/KokoEatingBanana.md`
- `padho-with-pratyush/Heap/task-scheduler.md`
- `padho-with-pratyush/Recursion/generate-parentheses.md`
- `padho-with-pratyush/Tree/validate-binary-search-tree.md`

This folder usually contains fuller explanations than the most compact notes, but the style still varies file to file.

## Important Reality About Style

This repository is useful, but not fully standardized.

An agent must work with that reality instead of pretending it is already normalized.

Observed style traits:

- Many files begin with `Prob:` and a direct LeetCode link.
- Many solutions are labeled like `Sol 1: Brute Force`, `Sol 2: Optimal`, or similar.
- Java is the dominant implementation language in problem notes.
- Some pattern files use polished Markdown headings and narrative sections.
- Some files use informal phrasing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thezaidsheikh/dsa-practice](https://github.com/thezaidsheikh/dsa-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
