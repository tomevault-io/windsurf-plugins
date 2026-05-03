---
trigger: always_on
description: Each implementation round must produce an implementation report Markdown file (YAML frontmatter + body) in the round evidence directory:
---


# Implementation Report Markdown is required per round

## Body
Each implementation round must produce an implementation report Markdown file (YAML frontmatter + body) in the round evidence directory:
- Include summary, changed files, automation outputs, Context7 packages queried, follow-ups, and blockers.
- Do not invent new report paths or filenames; follow the project’s configured schema/locations.

Reference: `guidelines/agents/OUTPUT_FORMAT.md`

---
id: RULE.PRISMA.ENUMS_FOR_CONSTRAINTS
title: Use Enums for Constrained Values
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/schema-design.md
---

# Use Enums for Constrained Values

## Body
Use enums for constrained value sets to ensure type safety.

---
id: RULE.PRISMA.EXPLICIT_RELATIONS
title: Explicit Relation Definitions
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/relationships.md
---

# Explicit Relation Definitions

## Body
Define explicit relations using @relation attribute for clarity and maintainability.

---
id: RULE.PRISMA.INDEXES_AND_CONSTRAINTS
title: Add Indexes and Unique Constraints
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/schema-design.md
---

# Add Indexes and Unique Constraints

## Body
Add unique constraints and indexes as needed for performance and data integrity.

---
id: RULE.PRISMA.MIGRATION_SAFETY
title: Safe Migration Practices
category: implementation
blocking: true
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/migrations.md
---

# Safe Migration Practices

## Body
Follow safe migration practices; review migrations before applying to production.

---
id: RULE.PRISMA.NORMALIZATION
title: Appropriate Schema Normalization
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/schema-design.md
---

# Appropriate Schema Normalization

## Body
Normalize appropriately; avoid excessive denormalization unless justified by performance requirements.

---
id: RULE.PRISMA.QUERY_OPTIMIZATION
title: Query Optimization Patterns
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/query-optimization.md
---

# Query Optimization Patterns

## Body
Follow Prisma query optimization patterns for efficient database access.

---
id: RULE.PRISMA.UUID_PRIMARY_KEYS
title: Use UUID Primary Keys
category: implementation
blocking: false
contexts: []
origins:
- pack:prisma
source:
  file: packs/prisma/guidelines/includes/prisma/schema-design.md
---

# Use UUID Primary Keys

## Body
Use UUID primary keys; prefer explicit relations with @relation.

---
id: RULE.REACT.ACCESSIBILITY
title: Accessibility Requirements (WCAG 2.1 AA)
category: implementation
blocking: true
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/accessibility.md
---

# Accessibility Requirements (WCAG 2.1 AA)

## Body
All components must meet accessibility standards (WCAG 2.1 AA minimum): semantic HTML, ARIA labels, keyboard navigation.

---
id: RULE.REACT.COMPONENT_COMPOSITION
title: Component Composition Over Inheritance
category: implementation
blocking: false
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/component-design.md
---

# Component Composition Over Inheritance

## Body
Favor composition over inheritance; extract pure presentational pieces.

---
id: RULE.REACT.FORM_ACTIONS
title: Server Actions for Forms (React 19)
category: implementation
blocking: false
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/server-client-components.md
---

# Server Actions for Forms (React 19)

## Body
Prefer Server Actions with form elements over client-side data fetching in React 19.

---
id: RULE.REACT.HOOKS_PATTERNS
title: React Hooks Best Practices
category: implementation
blocking: false
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/hooks-patterns.md
---

# React Hooks Best Practices

## Body
Follow React hooks best practices and patterns.

---
id: RULE.REACT.NO_USE_EFFECT_FETCHING
title: Avoid useEffect for Data Fetching (React 19)
category: implementation
blocking: false
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/HOOKS.md
---

# Avoid useEffect for Data Fetching (React 19)

## Body
In React 19, prefer Server Components or use() hook over useEffect for data fetching to improve performance.

---
id: RULE.REACT.RULES_OF_HOOKS
title: Rules of Hooks (React 19)
category: implementation
blocking: true
contexts: []
origins:
- pack:react
source:
  file: packs/react/guidelines/includes/react/HOOKS.md
---

# Rules of Hooks (React 19)

## Body
Only call hooks at top level; no conditional hooks, hooks in loops, or in nested functions.

---
id: RULE.REACT.SEMANTIC_HTML
title: Use Semantic HTML
category: implementation
blocking: false
contexts: []

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leeroybrun/happy-stacks](https://github.com/leeroybrun/happy-stacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
