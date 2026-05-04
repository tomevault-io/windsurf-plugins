---
trigger: always_on
description: **ABSOLUTE REQUIREMENT**: ALL code must be extremely readable and maintainable. NO EXCEPTIONS.
---

# ilamy Calendar - AI Coding Instructions

## 🚨 CRITICAL RULES

### 🚨 CRITICAL: ALWAYS WRITE HUMAN-READABLE CODE

**ABSOLUTE REQUIREMENT**: ALL code must be extremely readable and maintainable. NO EXCEPTIONS.

**MANDATORY Code Readability Standards:**

- **Extract complex operations into descriptive variables** - Never use inline calculations or nested operations
- **Use meaningful variable names** - `targetEventStartISO` instead of `targetEvent.start.toISOString()`
- **Break down complex expressions** - Separate date calculations, object creation, and conditional logic
- **One operation per line** - Avoid chaining multiple operations in a single statement
- **Descriptive intermediate variables** - `existingExdates`, `updatedExdates`, `dayBeforeTarget`, `terminationDate`

**Examples of GOOD vs BAD code:**

```typescript
// ❌ BAD - Unreadable inline operations
const updatedBaseEvent = {
  ...baseEvent,
  exdates: [...(baseEvent.exdates || []), targetEvent.start.toISOString()],
}

// ✅ GOOD - Readable with extracted variables
const targetEventStartISO = targetEvent.start.toISOString()
const existingExdates = baseEvent.exdates || []
const updatedExdates = [...existingExdates, targetEventStartISO]
const updatedBaseEvent = {
  ...baseEvent,
  exdates: updatedExdates,
}
```

**STRICT ENFORCEMENT:**

- **NEVER write complex inline expressions** - Always extract to variables first
- **NEVER chain multiple operations** without intermediate variables
- **ALWAYS use descriptive variable names** that explain the purpose
- **ALWAYS separate logical steps** into individual, readable operations

### NEVER START/STOP DEV SERVER

**The development server is ALWAYS running. NEVER run `bun dev` or any server start/stop commands. Always assume hot reloading is active.**

### 🚨 CRITICAL: SHORT AND INFORMATIVE COMMIT MESSAGES

**ABSOLUTE REQUIREMENT**: ALL commit messages must be short and informative. NO LONG, VERBOSE COMMIT MESSAGES.

**MANDATORY Commit Message Standards:**

- **Keep it concise** - Maximum 100 characters for the subject line
- **Use present tense** - "Fix bug" not "Fixed bug" or "Fixes bug"
- **Be specific** - Describe what was changed, not why
- **No redundant words** - Avoid "update", "modify", "change" when the action is clear

**Examples:**

- ✅ GOOD: `Fix recurring event deletion scope`
- ✅ GOOD: `Add recurrenceId validation to EventForm`
- ✅ GOOD: `Remove unused RecurrenceHandler class refs`
- ❌ BAD: `Update the copilot instructions file to fix the conflicting information about recurrenceId usage`
- ❌ BAD: `Fixed an issue where the recurring event deletion was not working properly`

**CONVENTIONAL COMMIT TYPES:**

Use conventional commit prefixes for consistency and automated tooling:

- **feat**: New feature or functionality
  - `feat: Add drag-and-drop for recurring events`
  - `feat: Implement weekly view navigation`

- **fix**: Bug fix or correction
  - `fix: Resolve timezone shift in EXDATE handling`
  - `fix: Prevent infinite loop in RecurrenceEditor`

- **docs**: Documentation changes
  - `docs: Update RFC 5545 compliance guidelines`
  - `docs: Add recurring event examples`

- **refactor**: Code restructuring without behavior change
  - `refactor: Extract event validation logic`
  - `refactor: Simplify RRULE parsing functions`

- **test**: Adding or updating tests
  - `test: Add recurring event edge case coverage`
  - `test: Fix RecurrenceEditor test assertions`

- **style**: Code formatting, linting fixes
  - `style: Fix ESLint violations in EventForm`
  - `style: Apply Prettier formatting`

- **chore**: Build, tooling, dependency updates
  - `chore: Update rrule.js to latest version`
  - `chore: Configure TypeScript strict mode`

**STRICT ENFORCEMENT:**

- **NEVER write commit messages longer than 100 characters**
- **NEVER use past tense** - always present tense
- **NEVER explain the why** - focus on the what
- **ALWAYS be direct and actionable**
- **ALWAYS use conventional commit prefixes** - feat, fix, docs, refactor, test, style, chore

### 🚨 CRITICAL: STRICT iCALENDAR (RFC 5545) COMPLIANCE

**ABSOLUTE REQUIREMENT**: ALL calendar functionality MUST strictly adhere to iCalendar RFC 5545 standards. NO EXCEPTIONS, NO FALLBACKS, NO SHORTCUTS.

**MANDATORY iCalendar Standards:**

- **RECURRENCE-ID**: Use `recurrenceId` field ONLY for modified recurring event instances
- **UID**: Every event must have a globally unique `uid` field - all instances share same UID
- **RRULE**: Use RFC 5545 compliant RRULE patterns (FREQ, INTERVAL, COUNT, UNTIL, BYDAY)
- **EXDATE**: Exclude instances using ISO string dates in `exdates` array
- **NO CUSTOM SCHEMES**: Never create custom ID parsing or fallback mechanisms

**STRICT ENFORCEMENT:**

```typescript
// ✅ CORRECT: Strict iCalendar compliance for modified instances
if (targetEvent.recurrenceId) {
  // This is a modified instance - handle accordingly
  console.log('Processing modified recurring instance')
}

// Find base event by UID and RRULE presence (with fallback UID generation)
const targetUID = targetEvent.uid
const baseEvent = events.find(
  (e) => (e.uid || `${e.id}@ilamy.calendar`) === targetUID && e.rrule
)
```

**Generated Event Instances (Regular) Must Have:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kcsujeet/ilamy-calendar](https://github.com/kcsujeet/ilamy-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
