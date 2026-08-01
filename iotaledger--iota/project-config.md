---
trigger: always_on
description: This file guides agents writing or editing real-world how-to pages in this directory. It supplements the parent `docs/CLAUDE.md` (Diataxis rules, code-embedding patterns, frontmatter requirements) and `docs/content/developer/iota-notarization/audit-trails/CLAUDE.md` (Audit Trails–specific conventions). Everything in those files applies here; this file adds patterns specific to real-world scenario pages.
---

# Real-World How-To Pages — Style Guide

This file guides agents writing or editing real-world how-to pages in this directory. It supplements the parent `docs/CLAUDE.md` (Diataxis rules, code-embedding patterns, frontmatter requirements) and `docs/content/developer/iota-notarization/audit-trails/CLAUDE.md` (Audit Trails–specific conventions). Everything in those files applies here; this file adds patterns specific to real-world scenario pages.

## Purpose

Real-world pages are **how-to guides** that demonstrate a complete business scenario using the IOTA Audit Trails client packages. They show an experienced developer how to model a multi-party, role-scoped audit trail for a specific industry use case (e.g., customs clearance, clinical trials, digital product passports).

## Page structure

Every real-world how-to page follows this exact section order:

### 1. Frontmatter

```yaml
---
title: '<Scenario Name> - Audit Trails'
sidebar_label: '<Short Label>'
description: 'Real-world example demonstrating how to use IOTA Audit Trails to <one-line goal>.'
tags:
  - notarization
  - audit-trails
  - how-to
---
```

- `title` always ends with `- Audit Trails`.
- `sidebar_label` is a short form for the navigation sidebar.
- `description` starts with "Real-world example demonstrating how to use IOTA Audit Trails to…".
- Tags always include exactly `audit-trails` and `how-to`. No additional tags.

### 2. Imports

```mdx
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
```

Always import both, even if you think only one is needed — both are used for code tabs.

### 3. Title heading and introduction

```mdx
# <Same as frontmatter title>

This real-world example <one paragraph describing the scenario, the actors involved, and the key Audit Trails features demonstrated>.
```

The introduction is a single paragraph. It names the actors, summarises the workflow, and highlights the Audit Trails features in play (e.g., tag-scoped roles, write locking, time-constrained access).

### 4. Business Context

```mdx
## Business Context

<Industry context>. A blockchain-based audit trail provides:

- **<Benefit>**: <Explanation>
- **<Benefit>**: <Explanation>
- ...
```

- 3–5 bullet points.
- Each bullet starts with a **bold benefit label** followed by a colon and a one-sentence explanation.
- Focus on why a tamper-proof audit trail matters for this specific domain.

### 5. Field Usage Strategy

```mdx
## Field Usage Strategy

- **`immutable_metadata`**: <What identity data is stored>
- **`updatable_metadata`**: <What mutable status is tracked>
- **`record.data`**: <What event payloads contain>
- **`record.metadata`**: <What structured context looks like> (e.g., `"event:some_event"`)
- **`record.tag`**: <What categories are used> — list the tag names
```

Always cover all five fields in this exact order. Use inline code for field names and example values.

### 6. Role Design table

```mdx
### Role Design

| Role | Permissions | RoleTags | Holder |
|------|------------|----------|--------|
| `Admin` | Full administrative control | — | <who> |
| `<RoleName>` | <Permission list> | `"<tag>"` | <who> |
| ...  | ... | ... | ... |
```

- Always include an `Admin` row first.
- Use inline code for role names and tag strings.
- Use an em dash (`—`) when a column is not applicable (e.g., no RoleTags for Admin).
- The table may include an optional `Constraint` column if time-windowed or conditional access is relevant (see `clinical-trial.mdx`).

### 7. Prerequisites

```mdx
## Prerequisites

- A funded IOTA account
- Access to an IOTA network (testnet, devnet, or local)
- Audit Trails client packages installed
- Familiarity with [Role-Based Access Control](../explanations/role-based-access-control.mdx)<optional-extra-links>
```

Always include the first four bullets. Add links to additional explanation or how-to pages when the scenario uses features beyond basic RBAC (e.g., locking, tagged records).

### 8. Implementation Overview

```mdx
## Implementation Overview

### 1. <Step Title>

<One or two sentences describing what this step does and why.>

<CodeTabs />

### 2. <Step Title>

...
```

- Number each step sequentially (`### 1.`, `### 2.`, etc.).
- Step titles should be imperative verb phrases describing the goal (e.g., "Create the Trail", "Define Tag-Scoped Roles", "Lock the Trail After Clearance").
- Each step has a brief prose description followed by a code tab block.
- **3–7 steps** is the typical range. Fewer than 3 means the scenario is too simple for a real-world page; more than 7 means it should be split.

#### Code tab block pattern

Always use this exact structure for code tabs within implementation steps:

```mdx
<div className={'hide-code-block-extras'}>
<Tabs groupId="language" queryString>
<TabItem value="rust" label="Rust">

\`\`\`rust reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iotaledger/iota](https://github.com/iotaledger/iota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
