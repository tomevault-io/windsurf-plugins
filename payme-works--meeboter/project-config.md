---
trigger: always_on
description: This skill guides collaborative design through:
---

# Agent Rules

## Brainstorming Before Implementation (CRITICAL)

**ALWAYS USE THE SUPERPOWERS BRAINSTORMING SKILL BEFORE STARTING ANY TASK**

Before implementing any feature, fix, or change, you MUST use the `superpowers:brainstorming` skill:

```
Skill: superpowers:brainstorming
```

This skill guides collaborative design through:

1. **Understanding the idea** - Ask questions one at a time (prefer multiple choice) to refine requirements
2. **Exploring approaches** - Propose 2-3 different approaches with trade-offs and your recommendation
3. **Presenting the design** - Break design into 200-300 word sections, validating each before continuing
4. **Documentation** - Write validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`

**Key principles from the skill:**
- One question at a time, don't overwhelm
- Multiple choice questions preferred when possible
- YAGNI ruthlessly, remove unnecessary features
- Always explore 2-3 alternatives before settling
- Incremental validation of each design section

This applies to ALL tasks including: new features, bug fixes, refactoring, configuration changes, database migrations, and any code modifications.

## Rule Organization (CRITICAL)

**Each rule MUST be in its own dedicated file in `/rules`** - Never add rules directly to CLAUDE.md. This file only contains brief references to rules.

- **Create dedicated rule files** - Each topic gets its own file in `/rules` (e.g., `rules/URL_STATE.md`, `rules/API_PATTERNS.md`)
- **Brief references only in CLAUDE.md** - Only add a one-line reference in the Quick Reference table
- **Library-specific rules go in `/rules/libraries/`** - E.g., `rules/libraries/NUQS.md`

```bash
# ✅ CORRECT: Rule in dedicated file
rules/URL_STATE.md          # Contains full URL state patterns
CLAUDE.md                   # Just references: "URL_STATE.md | nuqs, search params, sort format"

# ❌ WRONG: Rule directly in CLAUDE.md
CLAUDE.md                   # Contains full URL state patterns inline
```

## Rule Lookup Protocol (MANDATORY)

**BEFORE starting ANY task, you MUST identify and read the relevant rule files.** This is not optional. Failing to consult rules leads to inconsistent code and rework.

### How to Use This Protocol

1. **Identify task type** from the trigger list below
2. **Read the referenced rule file(s)** using the Read tool BEFORE writing any code
3. **Apply the patterns** exactly as documented
4. **When in doubt, search rules/** - Use `Glob` or `Grep` to find relevant rules

### Task-Based Rule Lookup

#### When Working on AUTHENTICATION or API KEYS
**Triggers**: Auth flows, sessions, API keys, Better-Auth, OAuth, sign-in/sign-up
**MUST READ**:
- [rules/AUTHENTICATION.md](rules/AUTHENTICATION.md) - Better-Auth SDK, API key patterns, auth hooks

**Search for**: `Grep pattern="auth|api.key|session|oauth" path="rules/"`

#### When Working on REACT COMPONENTS or UI
**Triggers**: Creating components, dialogs, sheets, tables, forms, styling
**MUST READ**:
- [rules/COMPONENT_PATTERNS.md](rules/COMPONENT_PATTERNS.md) - Component organization, Sheet vs Dialog, z-index, tables
- [rules/CODE_STYLE.md](rules/CODE_STYLE.md) - JSX conventions, composition patterns, React keys
- [rules/libraries/NEXTJS.md](rules/libraries/NEXTJS.md) - "use client"/"use server" directives, component boundaries

**Search for**: `Grep pattern="component|dialog|sheet|table|jsx" path="rules/"`

#### When Working on DATA TABLES (TanStack React Table)
**Triggers**: Creating tables, table columns, table meta, row selection, filtering, useReactTable
**MUST READ**:
- [rules/libraries/REACT_TABLE.md](rules/libraries/REACT_TABLE.md) - Table meta types, column definitions, custom filters, row selection

**Search for**: `Grep pattern="useReactTable|TableMeta|ColumnDef|react-table" path="rules/"`

#### When Working on NAMING (Interfaces, Variables, Files)
**Triggers**: Naming interfaces, types, booleans, enums, files, components
**MUST READ**:
- [rules/NAMING_CONVENTIONS.md](rules/NAMING_CONVENTIONS.md) - Interface naming, boolean prefixes, dialog naming, **no Details/Info/Data suffixes**

**Search for**: `Grep pattern="naming|interface|boolean|enum" path="rules/"`

#### When Working on TESTS
**Triggers**: Writing E2E tests, unit tests, tRPC tests, browser tests
**MUST READ**:
- [rules/TESTING.md](rules/TESTING.md) - E2E patterns, Better-Auth testing, Playwright MCP

**Search for**: `Grep pattern="test|e2e|playwright|mock" path="rules/"`

#### When Working on ZOD SCHEMAS
**Triggers**: Creating Zod schemas, validation, tRPC input/output, z.enum, z.object, z.void
**MUST READ**:
- [rules/libraries/ZOD.md](rules/libraries/ZOD.md) - Zod best practices, enum validation, empty inputs

**Search for**: `Grep pattern="z\.|zod|schema" path="rules/"`

#### When Working on DATABASE (Drizzle ORM)
**Triggers**: Migrations, queries, schema changes, PostgreSQL operations
**MUST READ**:
- [apps/milo/src/server/database/](apps/milo/src/server/database/) - Schema definitions
- [apps/milo/drizzle/](apps/milo/drizzle/) - Migration files

**CRITICAL**: Before writing ANY database operations, check the schema at `apps/milo/src/server/database/schema/` to understand table structures.

**Search for**: `Grep pattern="drizzle|schema|migration|database" path="apps/milo/"`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Payme-Works) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
